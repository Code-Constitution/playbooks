# Auto-fix PR enablement — operator playbook

[![Pattern: safe-fix](https://img.shields.io/badge/Pattern-safe--fix_whitelist-10b981)](https://codeconstitution.com)

**Scenario:** you want Code Constitution™ to auto-draft fix PRs for mechanical violations (whitespace, JSON canonicalisation, missing newlines, deterministic key ordering) without auto-merging anything that requires judgement.

## What this playbook gets you

- Safe-fix whitelist drives auto-PR composition
- Anything off the whitelist opens a draft PR with reviewer ping (NO auto-merge)
- Per-installation kill-switch

## Step 1 — Enable per-install

In your CC dashboard at app.codeconstitution.com/&lt;install&gt;/settings:

1. Toggle **Auto-fix PR composer** → ON
2. (Optional) Tick **BYO LLM** if you want LLM-assisted drafting for non-mechanical fixes — still human-approved, never auto-merged
3. Save

The setting is per-installation; you can enable on one repo and not another.

## Step 2 — Review what's on the safe-fix whitelist

The whitelist is deterministic-only. NOT on the whitelist:
- Logic changes
- Dependency updates
- Configuration changes
- Anything that needs the engineer's judgement

ON the whitelist:
- Trailing whitespace removal
- JSON canonicalisation (sorted keys)
- Missing newline-at-EOF
- Deterministic key ordering in YAML
- Lint-only auto-fixes from the rule packs

The exact whitelist for your installation is published at
`https://api.codeconstitution.com/v1/policy/safe-fix-whitelist` so you
can audit it before enabling.

## Step 3 — Configure your branch + review policy

CC opens auto-fix PRs as **drafts**. Branch protection should require:

- At least one human review on auto-fix PRs (treat them as any other PR)
- CC's own check-run passes (auto-fix PRs MUST pass their own engine run)
- Required statuses include the CC check

```yaml
# .github/branch-protection-rules.yml (or via the GitHub branch-protection API)
branches:
  main:
    required_pull_request_reviews:
      required_approving_review_count: 1
    required_status_checks:
      contexts: ["cc/verify"]
```

## Step 4 — What CC will + won't do

CC will:
- Open a draft PR titled `chore(cc): safe-fix mechanical violations for <head_sha>`
- Group fixes by file
- Include the exact rule-pack reference for each fix
- Sign the commit (GPG via the install token)

CC will NOT:
- Auto-merge anything
- Open PRs for non-whitelisted patterns
- Touch files outside the PR's diff scope
- Modify lockfiles, secrets, or generated artefacts

## Step 5 — BYO LLM for non-whitelisted fixes (optional)

If you want LLM-assisted drafting for non-mechanical fixes (still
human-approved, never auto-merged), configure your provider in the
dashboard:

- Provider: DeepSeek / Gemini / OpenAI / Anthropic / your-own-endpoint
- Key: held in your own KMS; CC never reads it
- Scope: limited to the engine's prompt; never sees full repo content

The LLM proposes a patch; the safe-fix whitelist still gates which
patches CAN open a PR. Non-whitelisted LLM patches stay as comments
on the original PR — never become commits.

## What the auditor will see

```
$ curl https://api.codeconstitution.com/v1/events \
    -d kind=safe_fix_pr_opened \
    -d tenant_id=<install_id>
```

Returns every auto-fix PR opened in the window, with the head_sha + rule-pack ref + reviewer who approved.

Apache-2.0.
