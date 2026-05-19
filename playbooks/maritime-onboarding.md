# Maritime sector pack — onboarding playbook

[![Sector: Maritime](https://img.shields.io/badge/Sector-Maritime-0ea5e9)](https://github.com/Compliance-to-Architecture/sector-packs)
[![Frameworks: 30](https://img.shields.io/badge/Frameworks-30-10b981)](https://github.com/Compliance-to-Architecture/sector-packs/tree/main/packs/maritime)

**Scenario:** you operate vessels (or vessel-supporting software) under IMO SOLAS / MARPOL / STCW / IACS UR E26 + E27 / BIMCO cyber / EU MRV.

## What this playbook gets you

- 30 maritime-specific frameworks pre-activated, crosswalked to ISO 27001 + SOC 2 so you don't double-instrument
- Flag-state, class-society, and port-state-control control overlays
- IACS UR E26/E27 cyber gates on engineering-system changes

## Step 1 — Activate the sector pack

Maritime is a **paid sector pack** (Enterprise tier). Once your tier is provisioned, drop this in `constitution.yaml`:

```yaml
version: 1
frameworks:
  - iso-27001     # free baseline
  - soc-2          # free baseline
sector_packs:
  - maritime       # 30 frameworks, paid
```

The next PR will activate the full pack and post per-control coverage status.

## Step 2 — Configure flag-state + class-society overlays

```yaml
sector_packs:
  - id: maritime
    flag_states: ["MH", "PA", "LR", "MT"]    # Marshall Islands, Panama, Liberia, Malta
    class_society: "DNV"                       # DNV, ABS, Lloyd's, BV, ClassNK, NK, CCS, KR, RINA, IRS, PRS
    port_state_controls: ["Paris MoU", "Tokyo MoU", "USCG"]
```

The flag-state and class-society lookups drive which authority documents the engine cites.

## Step 3 — Cyber gates (IACS UR E26 + E27)

Newbuild + retrofit cyber requirements:

- **E26**: Cyber resilience of ships
- **E27**: Cyber resilience of on-board systems

Add to `constitution.yaml`:

```yaml
risk_tiering:
  paths:
    - { glob: "engineering/main-engine/**",   tier: high-cyber, gates: [iacs-ur-e27] }
    - { glob: "engineering/nav-bridge/**",     tier: high-cyber, gates: [iacs-ur-e26] }
    - { glob: "engineering/cargo-control/**",  tier: high-cyber, gates: [iacs-ur-e27] }
```

PRs touching these paths require explicit IACS sign-off via the workflow.

## Step 4 — EU MRV reporting

EU Monitoring, Reporting + Verification for shipping emissions. The pack auto-generates the annual MRV report from your operational data feed.

```yaml
mrv:
  enabled: true
  verifier: "DNV-MRV-EU"      # any EU-accredited verifier
  vessel_imos: ["9876543", "9876544"]
```

## Step 5 — What the auditor / inspector will see

- **Class surveyor**: per-IACS-rule evidence pack URLs
- **Port State Control inspector**: structured CDG-PSC pack at /v1/events?source=maritime&kind=psc_pack
- **EU MRV verifier**: signed MRV report at `evidence/<head_sha>/mrv-annual.pack.json`
- **Flag-state administration**: per-flag-state compliance digest

## Cross-references

- Full framework catalogue: <https://github.com/Compliance-to-Architecture/sector-packs/tree/main/packs/maritime>
- BIMCO cyber security guidelines: cited in pack
- IMO MEPC.346(78): cited under MARPOL clauses

Apache-2.0.
