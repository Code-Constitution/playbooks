# EU AI Act readiness — operator playbook

[![Framework: EU AI Act](https://img.shields.io/badge/Framework-EU_AI_Act-0ea5e9)](https://github.com/Compliance-to-Architecture/framework)
[![Deadline: Aug 2026](https://img.shields.io/badge/High--risk_deadline-Aug_2026-ef4444)](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32024R1689)

**Scenario:** you ship AI features into the EU. The high-risk obligations enter into force in August 2026. You need a FRIA workflow, Annex IV technical documentation, and a CE-marked conformity assessment ready by then.

## What this playbook gets you

- Inline check-run on every PR that touches an AI system (model artefact, training data manifest, inference endpoint)
- Pre-merge FRIA gate: PRs that change human-impact AI flows cannot merge until a FRIA is on file
- Auto-drafted Annex IV technical documentation skeleton per model
- Signed evidence pack per inference-config change

## Step 1 — Install + enable

Install Code Constitution on the repo(s) where your AI lives. In the App settings → Repository access, add only what's in scope.

## Step 2 — `constitution.yaml`

Drop this at the root of each repo that ships AI:

```yaml
version: 1
frameworks:
  - eu-ai-act           # free
  - iso-42001           # free
  - nist-ai-rmf         # free
risk_tiering:
  default: limited
  paths:
    - { glob: "models/**/hiring*",        tier: high-risk, annex: III.4.a }
    - { glob: "models/**/credit-score*",  tier: high-risk, annex: III.5.b }
    - { glob: "models/**/biometric*",     tier: high-risk, annex: III.1.a }
fria:
  required_when: tier == "high-risk"
  template: eu-ai-act/art-27.fria
  signoff_roles: [dpo, ai-officer, product-owner]
annex_iv:
  generated_on_pr: true
  storage: byoc            # see byoc-vault-setup.md
artefacts_required:
  - model_card
  - data_card
  - human_oversight_plan
  - post_market_monitoring_plan
```

## Step 3 — Rule-pack activation evidence

Once the App reads this file, the next PR will:

1. Emit a check run: `cc / eu-ai-act` with the rule-pack-activation evidence
2. Open a tracking issue per high-risk path that lacks a FRIA
3. Refuse to merge any change to a high-risk path until the FRIA is signed off

## Step 4 — What the auditor will see

When the EU AI Act audit window opens, hand the auditor one URL per high-risk system:

```
https://<your-byoc-store>/cc/evidence/<repo>/<head_sha>/eu-ai-act-pack.json
```

The pack contains:

- `manifest.json` — control_refs → EU AI Act Art. {9,10,13,14,15,27} clause URIs
- `fria.pdf` — signed Fundamental Rights Impact Assessment
- `annex_iv.pdf` — technical documentation per Annex IV
- `conformity.json` — conformity-assessment checklist + CE-mark gate verdict
- `audit_trail.csv` — hash-chained event log for the head_sha
- `signatures.json` — sign-off identities + timestamps
- `framework_version.txt` — exact framework module version that produced this verdict (replay anchor)

The auditor verifies head-hash against the WORM chain; samples as deeply as their methodology requires; signs off. No email chain.

## Cross-references

- Framework spec: <https://github.com/Compliance-to-Architecture/framework>
- Sector pack — pharma AI (paid): see [`audit-prep-handover.md`](audit-prep-handover.md) for the activation step
- FRIA template: ships in the `eu-ai-act` rule pack — no manual download needed
