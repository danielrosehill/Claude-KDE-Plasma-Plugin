---
name: build-plasmoid-file
description: Use when the user wants to build a distributable `.plasmoid` archive of a KDE Plasma 6 widget for upload to store.kde.org / opendesktop.org. Triggers on phrases like "build a plasmoid file", "package this for the kde store", "create the .plasmoid", "make a release archive for opendesktop", "upload to kde-look". Bumps the version in `package/metadata.json`, zips the `package/` contents into a versioned `.plasmoid` file, and prints the upload checklist for the store product page.
---

# Build a `.plasmoid` File for Distribution

Plasma widgets are distributed on **store.kde.org** / **opendesktop.org** as a single archive named `<id>-<version>.plasmoid`. The `.plasmoid` file is just a zip of the contents of `package/` (no top-level wrapper directory). KDE's "Get New Plasma Widgets" pulls the file attached to the product's *Files* section — the GitHub repo is irrelevant to GHNS users.

## Layout assumption

The repo follows the standard KDE plasmoid layout:

```
<repo>/
  package/
    metadata.json            ← KPlugin.Version lives here
    contents/
      ui/, code/, config/
  install.sh                 ← used for local install (kpackagetool6)
```

If the repo doesn't have `package/metadata.json`, stop and tell the user — this isn't a Plasma 6 plasmoid in the standard layout.

## Steps

### 1. Read current state

- `package/metadata.json` → read `KPlugin.Name`, `KPlugin.Id`, `KPlugin.Version`.
- Run `git status --short` and `git log -1 --oneline` to confirm the working tree is clean (or warn the user if not).

### 2. Decide the new version

Ask the user **only if unclear**. Default rules:

- **Patch bump** (1.3.0 → 1.3.1) — bug fixes, packaging-only changes
- **Minor bump** (1.3.0 → 1.4.0) — new features, new config options, new UI
- **Major bump** (1.3.0 → 2.0.0) — breaking changes (config schema rewrites, etc.)

If the user gave an explicit version, use that verbatim. Otherwise propose one based on the recent commits (`git log <last-tag>..HEAD --oneline` if a tag exists) and confirm.

### 3. Bump the version

Edit `package/metadata.json` — change `KPlugin.Version` to the new value. Don't touch other fields.

### 4. Build the archive

The `.plasmoid` is a zip of `package/`'s **contents** (the metadata.json must be at the archive root, not inside a `package/` directory). Use a `(cd package && zip ...)` pattern:

```bash
cd <repo-root>
ID=$(jq -r '.KPlugin.Id' package/metadata.json)
VER=$(jq -r '.KPlugin.Version' package/metadata.json)
OUT="${ID}-${VER}.plasmoid"
rm -f "$OUT"
( cd package && zip -r "../$OUT" . -x '*.bak' '*~' '.DS_Store' )
ls -lh "$OUT"
```

Verify the archive:

```bash
unzip -l "$OUT" | head -20    # metadata.json should be at root, not inside package/
```

If `metadata.json` shows up as `package/metadata.json` in the listing, the archive is wrong and KDE won't accept it.

### 5. (Optional) Commit the version bump

If the working tree was clean before the bump, commit the metadata change:

```bash
git add package/metadata.json
git commit -m "Bump version to ${VER}"
```

Don't push automatically — let the user decide whether to push, tag, sync to OpenCode, etc. Just mention that the commit was made.

Don't commit the `.plasmoid` archive itself — it's a build artifact. Add it to `.gitignore` if not already excluded.

### 6. Print the upload checklist

After the file is built, surface this to the user (don't bury it in tool output):

> **Built `<ID>-<VER>.plasmoid`** — ready to upload.
>
> On the product page at opendesktop.org / store.kde.org:
> 1. Open **Files** → upload `<ID>-<VER>.plasmoid`
> 2. Update the product **version** to `<VER>`
> 3. Add a **Changelog** entry summarising what's new
> 4. Save / publish
>
> Existing GHNS users will see "Update available" the next time they open *Add Widgets → Get New Widgets*.

If you have a sense of what's new (from the recent commits), draft a 2–4 line changelog the user can paste in.

## Common pitfalls

- **Wrong archive shape**: If you `zip -r foo.plasmoid package/`, the archive contains `package/metadata.json` and KDE rejects it. Always `cd package && zip -r ../foo.plasmoid .`.
- **Missing `jq`**: fall back to `grep -oP '"Version":\s*"\K[^"]+' package/metadata.json` if jq isn't installed.
- **Filename convention**: use the `KPlugin.Id` (e.g. `com.danielrosehill.hebrewdate`) rather than the human name. Some users prefer a simpler name — ask if unsure, but the Id-based name is the safe default.
- **Pre-1.0 widgets**: the user may want `0.x` versions; respect that — don't force semver-major.

## When NOT to use

- If the user wants to install the widget locally — that's `install.sh` / `kpackagetool6 --upgrade`, not a `.plasmoid` build.
- If the user wants to publish to a debian package — that's a different pipeline.
- If the repo isn't a Plasma 6 plasmoid (no `package/metadata.json`) — stop and say so.
