# BYOC vault setup — operator playbook

[![Pattern: BYOC](https://img.shields.io/badge/Pattern-BYOC-10b981)](https://github.com/Compliance-to-Architecture/framework)
[![No-secrets-at-rest](https://img.shields.io/badge/Code_Constitution-holds_zero_credentials-0ea5e9)](https://codeconstitution.com)

**Scenario:** you want customer cloud signing material (AWS SNS signing certs, Azure Event Grid SAS keys, GCP Pub/Sub OIDC audiences, Cloudflare Logpush HMACs) to stay in YOUR vault. Code Constitution should be able to verify a signature without ever holding the key.

## What this playbook gets you

- The App proves `x` to itself by asking your vault: "did the customer sign this with their material?"
- Zero customer credentials at rest in the App
- One audit-trail row per verification, with `signature_verified: true|false` recorded

## Step 1 — Stand up the vault verifier endpoint

The App expects ONE endpoint:

```
POST  /verify?installation=<id>&kind=<provider_kind>
Headers: Authorization: Bearer <vault-token>
Body:    JSON with the per-provider canonical fields (see below)
Returns: 200 OK iff verification succeeds; non-2xx otherwise
```

Per-provider body shapes the App sends:

| Provider | `kind` | Body fields |
| --- | --- | --- |
| AWS SNS | `aws_sns_signature` | `signature_version`, `signature`, `signing_cert_url`, `topic_arn`, `message_id`, `subject`, `timestamp`, `message_sha256` |
| Azure Event Grid | `azure_event_grid_sas` | `presented` (the `aeg-sas-key` header value) |
| GCP Pub/Sub | `gcp_pubsub_oidc` | `token` (the JWT in `Authorization: Bearer`) |
| Cloudflare Logpush | `cf_logpush_hmac` | `presented` (the `cf-webhook-auth` header), `body_sha256` |

## Step 2 — Configure the App secrets

```
wrangler secret put CC_CUSTOMER_VAULT_LOOKUP_URL   # e.g. https://vault.your-company.com
wrangler secret put CC_CUSTOMER_VAULT_TOKEN        # bearer token the App uses
```

(Or store these via the GitHub App's secret store — the same names apply.)

## Step 3 — Wire your cloud's event bus to push

### AWS

```bash
# 1. Create an SNS topic for the events you want to send to CC
aws sns create-topic --name cc-cloud-events

# 2. Subscribe the CC endpoint
aws sns subscribe \
  --topic-arn arn:aws:sns:eu-west-1:123456789012:cc-cloud-events \
  --protocol https \
  --notification-endpoint https://api.codeconstitution.com/cloud-events/aws/<your-installation-id>

# 3. Wire CloudTrail / EventBridge to the topic
aws events put-rule --name cc-iam-changes \
  --event-pattern '{"source":["aws.iam"]}'
aws events put-targets --rule cc-iam-changes \
  --targets 'Id=cc,Arn=arn:aws:sns:eu-west-1:123456789012:cc-cloud-events'
```

CC handles the SNS subscription confirmation handshake automatically — you just have to subscribe.

### Azure

```bash
# 1. Create an Event Grid subscription
az eventgrid event-subscription create \
  --name cc-events \
  --source-resource-id <your-resource-id> \
  --endpoint https://api.codeconstitution.com/cloud-events/azure/<your-installation-id>

# 2. Pin the SAS key into your vault under kind=azure_event_grid_sas
```

CC handles the Event Grid validation handshake automatically.

### GCP

```bash
# 1. Create a Pub/Sub topic + push subscription
gcloud pubsub topics create cc-cloud-events
gcloud pubsub subscriptions create cc-push \
  --topic cc-cloud-events \
  --push-endpoint https://api.codeconstitution.com/cloud-events/gcp/<your-installation-id> \
  --push-auth-service-account cc-pubsub@<your-project>.iam.gserviceaccount.com
```

CC verifies the Google-signed OIDC token routed through your vault.

### Cloudflare

```bash
# 1. Set up Logpush with HMAC auth to the CC endpoint
curl -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE/logpush/jobs" \
  -H "Authorization: Bearer $CF_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name":"cc-logpush",
    "destination_conf":"https://api.codeconstitution.com/cloud-events/cloudflare/<install_id>?secret=<hmac>",
    "dataset":"http_requests"
  }'
```

## Step 4 — Verify

Trigger any controlled event in your cloud. Within 30 seconds (Constitution Amendment 2 budget):

1. Your event bus posts to `/cloud-events/<cloud>/<install>`
2. CC asks your vault to verify the signature
3. CC classifies the event into the 5-phase lifecycle
4. CC emits a reconciler event so the engine sees the state change
5. The event is indexed at `/v1/events?installation=<id>&source=cc-customer.<cloud>.*`

## What the auditor will see

```
$ curl https://api.codeconstitution.com/v1/events \
    -d installation=<id> \
    -d source=cc-customer.aws.cloudtrail \
    -d since=2026-05-01
```

Returns the full chain of customer-cloud events the App reacted to. Each carries `signature_verified: true|false`, the canonical 5-phase tag, and a pointer to the audit-trail row.

## Why this is the right pattern

- The App is stateless on customer credentials. A compromise of the App cannot exfiltrate signing material it never held.
- The vault is the only component that touches signing material — same blast radius whether you have 1 or 1,000 customers.
- Signature verification is testable independently of the App. Your vault can be audited as its own surface.
