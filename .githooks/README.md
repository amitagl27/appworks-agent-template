# Tracked Git hooks

This folder contains hooks that are versioned with the repo. Git does **not** use them automatically — you have to point Git at this directory once per clone:

```bash
git config core.hooksPath .githooks
```

After that, every contributor on every clone gets the same checks.

## What's here

### `pre-commit`

Rejects commits that touch AppWorks-owned files (`*.cws`, `.identifiers`, `*Base.java`, anything inside `#cws-ma#.cws/` or `#ef#/` folders) **unless** the committer's email matches the AppWorks sync identity.

The sync identity comes from (in priority order):

1. The `APPWORKS_SYNC_EMAIL` environment variable
2. `git config appworks.syncEmail`
3. A safe-by-default placeholder (`appworks-sync@example.com`) that no real user should match

Set option 2 per-repo:

```bash
git config appworks.syncEmail "appworks-sync@your-org.com"
```

## Why a hook AND a CI check?

The hook is convenience — fast local feedback so you catch mistakes before pushing. The CI workflow at `.github/workflows/guard-native-artefacts.yml` is the **enforcement** — it runs server-side where the developer can't disable it. Skipping the hook (`git commit --no-verify`) gets blocked at PR review.

Note: `.claude/settings.json` denies `git commit --no-verify` for Claude Code specifically.
