# Connecting an AppWorks PA workspace to this repository

Field-tested checklist. The gotchas at the bottom are the ones that cost real time.

## Prerequisites

- GitHub repo created from this template
- A GitHub Personal Access Token (PAT) with `Contents: Read & Write` and `Metadata: Read` on this repo
- An AppWorks Process Automation workspace ready to attach

## Step 1 — Activate Git LFS on your fork

**Do this before the first PA connection attempt**, even if you have no LFS-tracked files. PA's GitAdapter probes the LFS locks endpoint during `Test Connection`. GitHub returns HTTP 404 on that endpoint until at least one LFS object has been pushed to your repo, and the adapter surfaces the 404 as the misleading `Error: Not Found`.

GitHub does not allow LFS content in template repositories, so this template intentionally ships *without* LFS objects — every fork has to activate LFS on its own. The good news: it's three commands and a one-time push.

```bash
# In a fresh clone of your forked repo:
git lfs install
git lfs track "*.bin"

# Create a tiny placeholder so an LFS object actually gets pushed
echo "LFS activation $(date)" > lfs-placeholder.bin

git add .gitattributes lfs-placeholder.bin
git commit -m "Activate Git LFS"
git push origin main
```

You should see `Uploading LFS objects: 100% (1/1), …` in the push output. That's the signal LFS is now active server-side and PA's probe will return 200 instead of 404.

## Step 2 — Configure the PA workspace SCM connection

Open your PA workspace's SCM/Git configuration and fill in:

| Field | Value | Notes |
|---|---|---|
| URL | `https://github.com/<your-org>/<your-repo>.git` | **The `.git` suffix is required.** GitHub's LFS routing depends on it. Without it, PA's `isLFSEnabled` probe 404s and connection fails. |
| Username | your GitHub username | |
| Personal access token | the PAT from prerequisites | Pasted into the password field |
| Email | the sync identity, e.g. `appworks-sync@<your-org>.com` | This is the email PA will commit under. It must match the value of `APPWORKS_SYNC_EMAIL` in your repo Variables, or CI will reject every PA commit. |
| Branch | `main` | PA commits to a single branch. |
| Proxy | usually disabled | Enable only if your network requires it. |

## Step 3 — Test the connection

Click *Test Connection*. Expected: success message.

If you get `SOAP:Fault → Error: Not Found` with a stack trace containing `GitLockRESTHandler.isLFSEnabled`:

1. **Check the URL ends in `.git`.** This is the #1 cause.
2. **Verify LFS is active** with `git lfs ls-files` on a local clone — should list at least `lfs-placeholder.bin`.
3. **Re-run** the LFS push from Step 1.

Other failure modes:

| Symptom | Likely cause | Fix |
|---|---|---|
| `403 Forbidden` | PAT lacks `Contents: Read & Write` | Regenerate the PAT with correct scope |
| `401 Unauthorized` | PAT expired or username mismatch | Check both fields; PATs have expiry dates |
| `Cannot resolve host` | Network/DNS / proxy issue | Configure proxy in the PA SCM config |

## Step 4 — Confirm the first commit round-trip

In the AppWorks Web IDE, make a trivial change (rename a description, add a folder). Commit from the IDE. Then on your local clone:

```bash
git pull
git log -1
```

You should see PA's commit with `Author: <sync email>`. CI runs on the push and passes because the sync identity matches the configured `APPWORKS_SYNC_EMAIL`.

## Step 5 — Confirm the guardrails are working

From your developer identity (not the sync identity), try to commit a `.cws` change:

```bash
git config user.email "dev@your-org.com"   # your real dev identity
echo "x" >> some/path/AnyFile#cws-bpm#.cws  # any tracked .cws file
git add .
git commit -m "test"
# Should be rejected by the pre-commit hook
```

If the hook activated and rejected the commit, the four enforcement layers are functioning. Revert the change (`git restore .`) and you're done with setup.

## Operating-mode notes

- **Branch strategy**: PA writes to whichever branch you configured. To do "developer branches", developers branch off `main` and merge back via PR. PA itself does not understand branches in a per-developer sense.
- **Translations**: If two team members translate the same string in the same locale, they'll collide on the same `#cws-trt#.cws` file. The mergeable-association layout shards per-string, not finer.
- **`.identifiers` files**: PA regenerates these. Don't be alarmed when they show up in your diff — they're system-generated and should be left alone.
