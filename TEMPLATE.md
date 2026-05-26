# Using this template

## What you get when you fork

```
your-fork/
├── CLAUDE.md                        ← Read this first (so will your AI agent)
├── README.md                        ← User-facing overview
├── TEMPLATE.md                      ← This file
├── .gitignore
├── .gitattributes                   ← (you'll add Git LFS rules here per docs/connecting-pa.md)
├── .claude/
│   └── settings.json                ← Claude Code permission deny rules
├── .githooks/
│   ├── pre-commit                   ← Local commit guard
│   └── README.md
├── .github/
│   └── workflows/
│       └── guard-native-artefacts.yml   ← CI enforcement
├── docs/
│   ├── file-type-glossary.md
│   ├── pattern-traditional.md
│   ├── pattern-entity.md
│   └── connecting-pa.md
└── examples/
    ├── traditional/                 ← Sanitized fragments mirroring a real project
    └── entity/                      ← Sanitized fragments mirroring a real entity project
```

## Step-by-step setup

### 1. Fork via GitHub UI

Click **Use this template → Create a new repository** on GitHub. Make it private if it'll hold real project work.

### 2. Clone and activate hooks

```bash
git clone https://github.com/<your-org>/<repo>.git
cd <repo>
git config core.hooksPath .githooks
```

### 3. Set the AppWorks sync identity

Decide on the email AppWorks PA will commit under (e.g., `appworks-sync@<your-org>.com`). This is **the** identity that's allowed to commit `.cws` files. Two places to set it:

**Local (.git/config):** when *PA* runs locally for your dev workspace, configure PA to push as that identity.

**CI (GitHub repo Settings → Secrets and variables → Variables):**
```
APPWORKS_SYNC_EMAIL = appworks-sync@<your-org>.com
```

### 4. Generate a GitHub PAT for PA

PA needs a Personal Access Token to push:

- **Fine-grained PAT** (recommended), scoped to this repo only
- Permissions: **Contents: Read & Write**, **Metadata: Read**
- Expiration: 90 days (rotate before expiry)

### 5. Drop the AppWorks Platform Javadocs at `docs/api-reference/`

**This is the difference between an AI agent that writes correct AppWorks Java and one that writes plausible-looking nonsense.** AppWorks's `com.cordys.*` and `com.eibus.*` APIs (CustomBusObject, Node, Connector, XPath helpers, etc.) are not represented in any modern LLM's training data at meaningful volume. Without an on-disk reference, the agent will guess at method names and parameter orders, and the resulting code will compile but fail at runtime.

- Download the Javadocs for your AppWorks Platform version from **OpenText My Support** (search "AppWorks Platform Javadocs" — they ship as a ZIP).
- Unzip the contents into `docs/api-reference/` so that `docs/api-reference/index.html` and `docs/api-reference/allclasses-index.html` exist.
- The folder is gitignored (OT copyright forbids redistribution) — every developer on the team does this once per machine.

The `CLAUDE.md` file in this template instructs the agent to consult `docs/api-reference/` before writing any Java that imports `com.cordys.*` or `com.eibus.*`. If the folder is missing, the agent is instructed to stop and ask the user to install the Javadocs rather than guess.

### 6. Connect AppWorks PA workspace

In your PA workspace SCM config:

| Field | Value |
|---|---|
| URL | `https://github.com/<your-org>/<repo>.git` *(the `.git` suffix is mandatory)* |
| Username | your GitHub username |
| Personal access token | the PAT from step 4 |
| Email | the sync identity email |
| Branch | `main` |

Click **Test Connection**. It should pass. If you see `Error: Not Found` with an `isLFSEnabled` stack frame, you skipped the LFS activation in step 1 of [`docs/connecting-pa.md`](docs/connecting-pa.md) — go back and run those three commands first.

### 7. Verify the guardrails work

Try (as a sanity check):

```bash
# Should be rejected by the pre-commit hook
git config user.email "dev@example.com"
echo "x" >> some/path/Anything#cws-bpm#.cws
git add . && git commit -m "test"
# ↑ this commit should fail
```

Then revert and try a `.java` file change with your dev identity — that should succeed.

## What to customize after forking

1. **`README.md`** — replace the description with your actual project context
2. **`CLAUDE.md`** — read it carefully; add project-specific safe paths if your project lives in unusual locations
3. **`.claude/settings.json`** — if your AI agent isn't Claude Code, replicate equivalent deny patterns in your tool's config
4. **`examples/`** — delete this folder once you've internalized the patterns; it ships only as reference material

## Companion article

The full story — including the LFS gotcha, the URL-must-end-in-`.git` gotcha, the Java `Base.java` generated/extendable pattern, and a worked example for each project type — is in this LinkedIn article:

> *I Pair-Programmed With Claude on OpenText AppWorks for Two Months. Here's the Setup Nobody in the AppWorks Developer Community Has Tried*
>
> *(link added on publish)*
