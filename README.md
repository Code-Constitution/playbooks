# Code Constitution™ — Operator Playbooks

[![Apache-2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![GitHub App](https://img.shields.io/badge/Install-GitHub_App-181717?logo=github)](https://github.com/apps/code-constitution)
[![Live](https://img.shields.io/badge/Site-codeconstitution.com-0ea5e9)](https://codeconstitution.com)
[![Framework](https://img.shields.io/badge/Spec-Compliance--to--Architecture-10b981)](https://github.com/Compliance-to-Architecture/framework)

Operator-grade playbooks for using **Code Constitution™** in production. Each playbook is end-to-end runnable: a real scenario, the exact App settings, the exact `constitution.yaml`, the exact evidence the auditor will see.

## Playbooks

| Playbook | When to use |
| --- | --- |
| [eu-ai-act-readiness.md](playbooks/eu-ai-act-readiness.md) | You ship AI features in the EU and need to be ready for the August 2026 high-risk obligations + FRIA workflow |
| [maritime-onboarding.md](playbooks/maritime-onboarding.md) | You operate vessels under IMO SOLAS / MARPOL / IACS UR E26-27 and want the Maritime sector pack on your repos |
| [byoc-vault-setup.md](playbooks/byoc-vault-setup.md) | You need customer cloud credentials to stay in your own vault (the App never sees signing material) |
| [auto-fix-pr-enablement.md](playbooks/auto-fix-pr-enablement.md) | You want safe-fix mechanical patches as auto-drafted PRs |
| [audit-prep-handover.md](playbooks/audit-prep-handover.md) | You're 30 days from an audit window — what to hand the auditor + how |

## Conventions

- Every playbook ships a `constitution.yaml` snippet you can copy into your repo
- Every playbook lists which framework rule packs activate
- Every playbook names the evidence kinds produced + where they end up
- Every playbook ends with "what the auditor will see"

## Licence

Apache-2.0. The **Code Constitution™** trademark is not licensed.
