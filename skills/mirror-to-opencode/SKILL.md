---
name: mirror-to-opencode
description: Use when the user wants to mirror or sync a GitHub-hosted plasmoid (or any repo) to an OpenCode mirror repo for wider distribution. Triggers on phrases like "sync to opencode", "mirror to opencode", "push this to opencode", "update the opencode copy", "distribute this widget on opencode". Handles first-time clone of the OpenCode counterpart, rsync of source files (excluding `.git` and the sync helper itself), and a commit + push on the OpenCode side.
---

# Mirror a GitHub Repo to OpenCode

OpenCode is a separate git host (e.g. `git@opencode.net:<user>/<repo>.git`) the user maintains as a distribution channel parallel to GitHub. This skill keeps the OpenCode copy in sync with the canonical GitHub working tree.

## Layout assumptions

- **Canonical source** — `~/repos/github/my-repos/<Repo-Name>/` (Train-Case)
- **OpenCode mirror** — `~/repos/other-networks/opencode/<repo-name>/` (kebab-case, lower-case)
- The mirror has its own `.git/` and its own `origin` pointing at the OpenCode host.
- The mirror is **not** a git fork — it's an rsync'd copy. History is independent.

If those paths don't exist on the user's machine, ask before creating them.

## When to use

- The user shipped a plasmoid (or any project) on GitHub and wants OpenCode users to have the latest build.
- The canonical repo just got a release, version bump, README update, or new screenshot, and the OpenCode mirror is now stale.
- The user is setting up the OpenCode mirror for the first time.

## Operation

### First-time setup

1. Confirm the OpenCode mirror repo exists on the OpenCode host. If not, the user creates it via OpenCode's web UI — this skill does not auto-create remote repos there.
2. Clone it into `~/repos/other-networks/opencode/<repo-name>/`:
   ```bash
   git clone git@opencode.net:<user>/<repo-name>.git ~/repos/other-networks/opencode/<repo-name>
   ```
3. Proceed to the sync step below.

### Routine sync (canonical → mirror)

```bash
SOURCE=~/repos/github/my-repos/<Repo-Name>
DEST=~/repos/other-networks/opencode/<repo-name>

rsync -av --delete \
    --exclude='.git' \
    --exclude='sync-to-opencode.sh' \
    --exclude='.github' \
    "$SOURCE/" "$DEST/"
```

Key flags:

- `--delete` keeps the mirror a true reflection of the source — files removed in the canonical repo are removed in the mirror.
- `--exclude='.git'` is **mandatory** — the mirror has its own git history.
- `--exclude='sync-to-opencode.sh'` keeps the mirror clean of the maintenance script (which is GitHub-side only).
- `--exclude='.github'` skips GitHub-specific workflows that won't run on OpenCode.

Then commit and push from the mirror:

```bash
cd "$DEST"
git add -A
if ! git diff --cached --quiet; then
    git commit -m "Sync from upstream $(date -u +%Y-%m-%d)"
    git push
else
    echo "No changes to mirror."
fi
```

### Optional: install/refresh the helper script

If the canonical repo doesn't already have a `sync-to-opencode.sh`, offer to drop one in so the user can run the sync without invoking the skill. Template:

```bash
#!/bin/bash
SOURCE_DIR="<absolute path to canonical repo>"
DEST_DIR="<absolute path to opencode mirror>"

rsync -av --delete \
    --exclude='.git' \
    --exclude='sync-to-opencode.sh' \
    --exclude='.github' \
    "$SOURCE_DIR/" "$DEST_DIR/" || { echo "rsync failed"; exit 1; }

cd "$DEST_DIR" && git add -A
if ! git diff --cached --quiet; then
    git commit -m "Sync from upstream $(date -u +%Y-%m-%d)"
    git push
else
    echo "No changes to mirror."
fi
```

`chmod +x sync-to-opencode.sh` after writing.

## Safety rules

- Never run `--delete` rsync into a path that isn't a known OpenCode mirror — confirm the destination has an OpenCode `origin` first:
  ```bash
  git -C "$DEST" remote get-url origin | grep -q opencode || { echo "Not an OpenCode mirror — refusing"; exit 1; }
  ```
- Never push from the mirror without committing — `git push` on a dirty tree silently leaves uncommitted changes behind.
- Don't sync when the canonical tree has uncommitted changes the user hasn't pushed to GitHub yet — mirror what's published, not what's local. Check `git -C "$SOURCE" status --porcelain` first; if non-empty, ask the user whether to proceed.

## Output convention

When asked to sync:

1. Resolve canonical and mirror paths (ask if unclear).
2. Run the safety check on the destination's `origin`.
3. Run the canonical-tree cleanliness check; warn if dirty.
4. Run the rsync.
5. Show the user the diff (`git -C "$DEST" status` + `git -C "$DEST" diff --stat HEAD`) before committing.
6. Commit and push, or report "no changes" if the mirror was already current.
