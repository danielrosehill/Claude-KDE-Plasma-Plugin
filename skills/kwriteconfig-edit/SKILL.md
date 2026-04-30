---
name: kwriteconfig-edit
description: Safely edit KDE config files via kwriteconfig6 / kwriteconfig5 with backup-before-write. Use whenever you need to set a key in a KDE rc file (kwinrc, kdeglobals, plasmarc, etc.) — never edit those files with sed.
---

# kwriteconfig safe edit

KDE config files use a custom INI dialect with array suffixes (`[$e]`, `[$i]`), nested groups (`--group A --group B`), and comments KDE will rewrite. **Never** use `sed` / `awk` / hand-rolled INI parsers on them — use `kwriteconfig6` (or `kwriteconfig5` on KF5).

## Detection

```bash
KW=$(command -v kwriteconfig6 || command -v kwriteconfig5)
KR=$(command -v kreadconfig6 || command -v kreadconfig5)
```

## Read a key

```bash
$KR --file kdeglobals --group General --key ColorScheme
```

## Set a key (with backup)

```bash
TARGET="$HOME/.config/kwinrc"
[ -f "$TARGET" ] && cp -a "$TARGET" "$TARGET.bak.$(date +%s)"
$KW --file kwinrc --group Compositing --key Backend "OpenGL"
```

## Nested groups

```bash
$KW --file plasma-org.kde.plasma.desktop-appletsrc \
    --group Containments --group 1 --group General \
    --key wallpaperplugin "org.kde.image"
```

## Delete a key / group

```bash
$KW --file kwinrc --group Compositing --key Backend --delete
$KW --file kwinrc --group Compositing --delete-group
```

## After-edit reload

Different components watch different files. Common ones:

| File | Reload |
|---|---|
| `kwinrc` | `qdbus org.kde.KWin /KWin reconfigure` |
| `kglobalshortcutsrc` | `qdbus org.kde.kglobalaccel /kglobalaccel reloadConfiguration` |
| `kdeglobals` | most apps re-read on launch; `kbuildsycoca6` for menu/icon caches |
| `plasma-org.kde.plasma.desktop-appletsrc` | restart `plasmashell` |
| `khotkeysrc` | `qdbus org.kde.kded6 /modules/khotkeys reread_configuration` |

## Notes

- Backups go to `${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/kde-plasma/state/kwriteconfig-backups/` (preferred over inline `.bak` clutter for non-trivial sessions).
- Boolean values are lowercase: `true` / `false`.
