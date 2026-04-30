---
name: kwin-scripting
description: List, install, load, unload, and remove KWin scripts on Plasma 6 / Plasma 5. Use when the user wants to manage window-manager scripts (tiling helpers, focus-follows-mouse tweaks, window-rule scripts, KWin effects bundled as scripts).
---

# KWin Scripting

Manage scripts that extend the KWin window manager.

## Tooling

- **Plasma 6 (Wayland)**: `kpackagetool6 --type=KWin/Script ...` and `qdbus6 org.kde.KWin /Scripting`.
- **Plasma 5**: `kpackagetool5` and `qdbus org.kde.KWin /Scripting`.

Detect which is in use:

```bash
if command -v kpackagetool6 >/dev/null; then KP=kpackagetool6; QD=qdbus6; else KP=kpackagetool5; QD=qdbus; fi
```

## Common operations

### List installed scripts

```bash
$KP --type=KWin/Script --list
```

### Install a script (.kwinscript file)

```bash
$KP --type=KWin/Script --install /path/to/script.kwinscript
```

### Remove

```bash
$KP --type=KWin/Script --remove <pluginName>
```

### Enable / disable

KWin scripts are enabled in `~/.config/kwinrc` under `[Plugins]` keys named `<pluginName>Enabled=true|false`. Use the `kwriteconfig-edit` skill for safe edits, then reconfigure KWin:

```bash
$QD org.kde.KWin /KWin reconfigure
```

### Load a one-shot script (without installing)

```bash
$QD org.kde.KWin /Scripting loadScript /path/to/script.js
$QD org.kde.KWin /Scripting/Script<N> run
```

## Notes

- After install/remove, reload KWin's script registry: `$QD org.kde.KWin /KWin reconfigure`.
- Logs: `journalctl --user -u plasma-kwin_wayland.service -f` or `~/.xsession-errors` on X11.
- Daniel's data dir for any scratch scripts: `${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/kde-plasma/data/kwin-scripts/`.
