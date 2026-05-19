# Audit prep handover — operator playbook

[![Audit-ready](https://img.shields.io/badge/Auditor-URL_handover-10b981)](https://codeconstitution.com)
[![WORM](https://img.shields.io/badge/Audit_trail-WORM_hash--chained-0ea5e9)](https://github.com/Compliance-to-Architecture/framework)

**Scenario:** your audit window opens in 30 days. The auditor has asked for evidence by sample-id. You want to hand them URLs, not folders.

## What this playbook gets you

- Per-framework evidence pack URLs to email the auditor on day 1
- Replay anchor (head_sha + framework version) so they can reconstruct any verdict
- A pre-flight gap report — controls that lack evidence in the last 90 days

## Step 1 — Generate the per-framework evidence index

```bash
# Replace <install_id> + <framework> with yours
curl https://api.codeconstitution.com/v1/events?source=evidence-event-agent \
  -d kind=evidence_pack_sealed \
  -d tenant_id=<install_id> \
  -d entity=urn:framework:<framework> \
  -d since=$(date -u -d "90 days ago" +%Y-%m-%dT%H:%M:%SZ) \
  -d limit=500
```

Returns one row per sealed evidence pack. Each row carries `payload_uri` — the URL the auditor opens.

## Step 2 — Build the auditor handover

Compose one email per framework. Include:

```
Subject: <Org> SOC 2 evidence handover — head_sha a1b2c3d, packs 90-day

The evidence index for our SOC 2 audit window is below. Each line is one
sealed pack; the head_sha + framework_version are sufficient to replay
the verdict.

Pack id                            Control refs              Sealed at
a1b2…   CC6.1 · CC7.2 · CC8.1     2026-05-12T10:14:00Z
b2c3…   CC6.1                      2026-05-13T08:22:00Z
…

Hash-chain head: <head_hash>
Framework version pinned: regunav-soc2@1.2.3

To verify a pack, recompute sha256 of the canonical-JSON payload and
match against the pack's payload_sha256 field.
```

## Step 3 — Pre-flight gap check

Before the auditor opens the packs, run:

```bash
curl https://api.codeconstitution.com/v1/admin/coverage \
  -d framework=<framework> \
  -d tenant=<install_id> \
  -d window_days=90 \
  | jq '.controls | map(select(.evidence_count == 0))'
```

Returns the controls with no evidence in window. Fix BEFORE the auditor
notices.

## Step 4 — What the auditor will see

For each pack URL:

1. Open in browser → JSON pack downloads
2. Verify `payload_sha256` against the file's actual hash
3. Verify `manifest_sha256` against `canonicalJson(manifest)` SHA
4. Cross-reference `head_sha` with the framework-version pin
5. If both hashes match: the pack is intact and the verdict is replayable

No email chain. No screenshots. No "we'll get back to you on Tuesday".

## Common findings

- **Evidence staleness** — a pack older than the control's
  evidence-frequency (per `@regunav/dictionaries/evidence-frequency`)
  is flagged. Refresh by re-running the controlled change.
- **Missing FRIA** — for EU AI Act high-risk paths. See
  [eu-ai-act-readiness.md](eu-ai-act-readiness.md).
- **Hash-chain break** — extremely rare; means the WORM store was
  tampered with. Security incident.

Apache-2.0.
