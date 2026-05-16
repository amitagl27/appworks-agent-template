# appworks-agent-template

A starter repo for **OpenText AppWorks / Process Automation (PA)** projects that lets AI coding agents (Claude Code, Cursor, Aider, etc.) work safely alongside the AppWorks Web IDE in the *same* Git repository.

## What this solves

AppWorks/PA workspaces sync to Git as a forest of XML files with names like `MyProcess#cws-bpm#.cws` and folders like `LegalCase#ef#`. These files are **authored exclusively by AppWorks** — any edit from outside the Web IDE corrupts the project model.

The problem: AI coding agents don't know which files are off-limits. By default, an agent asked to "refactor the email logic" will happily rewrite a `.cws` XML file and break the workspace.

This template encodes the off-limits set in **four enforcement layers**:

| Layer | File | What it does |
|---|---|---|
| Documentation | `CLAUDE.md` | Tells AI agents (and humans) which files are owned by AppWorks |
| Tool permissions | `.claude/settings.json` | Hard-denies Claude Code from editing protected paths |
| Local hook | `.githooks/pre-commit` | Rejects commits that touch protected files from non-sync identities |
| CI guardrail | `.github/workflows/guard-native-artefacts.yml` | Same check, on every PR/push — the rule an agent cannot bypass |

## Quick start

```bash
# 1. Use this template (GitHub: "Use this template" button)
# 2. Clone your new repo
git clone https://github.com/<your-org>/<your-repo>.git
cd <your-repo>

# 3. Activate the tracked Git hooks
git config core.hooksPath .githooks

# 4. Set the AppWorks sync identity for CI (your repo settings → Variables)
#    APPWORKS_SYNC_EMAIL = the email PA commits under

# 5. Connect your AppWorks PA workspace to this repo
#    See: docs/connecting-pa.md
```

## Two project patterns supported

AppWorks projects fall into two structural patterns. This template covers both:

- **Traditional pattern** — folders like `BPM/`, `JS/`, `CSS/`, `DB Metadata/`, `Webservices/`. Lots of agent-editable Java/JS/CSS surface inside the same repo. See [`docs/pattern-traditional.md`](docs/pattern-traditional.md).
- **Entity-based pattern** — folders like `Entities/`, `Layouts/`, `Processes/`. Almost everything is PA-managed XML. Agents work in a sibling repo or external service. See [`docs/pattern-entity.md`](docs/pattern-entity.md).

The file-type glossary in [`docs/file-type-glossary.md`](docs/file-type-glossary.md) explains every `#cws-XXX#` and `#XXX#` marker you'll see in committed files.

## Background

This template was built and documented based on real OpenText AppWorks Process Automation projects. The story behind it — including the painful Git LFS / `.git`-suffix gotchas you'll hit during your first `testConnection` — is in the companion LinkedIn article (link in `TEMPLATE.md`).
