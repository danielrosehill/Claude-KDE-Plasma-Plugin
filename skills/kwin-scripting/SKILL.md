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
ID=$($QD org.kde.KWin /Scripting org.kde.kwin.Scripting.loadScript /path/to/script.js my-oneshot)
$QD org.kde.KWin /Scripting/Script$ID org.kde.kwin.Script.run
$QD org.kde.KWin /Scripting org.kde.kwin.Scripting.unloadScript my-oneshot
```

Verified on Plasma 6.6.6 Wayland, 2026-10-19:

- Pass a plugin name and unload by it afterwards. Reloading the same name while it is
  still loaded fails.
- `loadScript` is overloaded `(s)` and `(ss)`. `qdbus6` resolves it. dbus-python's
  introspecting proxy picks `(s)` and fails with *"Fewer items found in D-Bus signature
  than in Python arguments"*. Call it with an explicit `"ss"` signature
  (`bus.call_blocking(..., "loadScript", "ss", (path, name))`).
- `print()` output appears in the user journal as `js: <text>`. Read it with
  `journalctl --user --since "-10s" -o cat | grep <marker>`. There is no D-Bus return
  value, so put a unique marker in what you print.

## Notes

- After install/remove, reload KWin's script registry: `$QD org.kde.KWin /KWin reconfigure`.
- Logs: `journalctl --user -u plasma-kwin_wayland.service -f` or `~/.xsession-errors` on X11.
- Daniel's data dir for any scratch scripts: `${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/kde-plasma/data/kwin-scripts/`.
