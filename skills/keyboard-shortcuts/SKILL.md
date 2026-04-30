---
name: keyboard-shortcuts
description: Inspect and edit KGlobalAccel keyboard shortcuts (global hotkeys) on KDE Plasma. Use when the user wants to bind, unbind, or audit a Meta/Ctrl/Alt key combination, or troubleshoot a shortcut that stopped firing.
---

# KGlobalAccel Shortcuts

Plasma global shortcuts are stored in `~/.config/kglobalshortcutsrc` and managed by the `kglobalaccel` daemon.

## Inspect

```bash
# List all shortcuts grouped by component
grep -E '^\[|^[A-Za-z]' ~/.config/kglobalshortcutsrc | less

# What is bound to a key combo? (D-Bus)
qdbus org.kde.kglobalaccel /kglobalaccel org.kde.KGlobalAccel.allMainComponents
```

## Edit a shortcut

Format in `kglobalshortcutsrc`:

```
ShortcutName=<active>,<default>,<friendly name>
# e.g.
Activate Krunner=Alt+Space,Alt+Space,Search
```

- `<active>=none` to unbind.
- Two comma-separated combos can be set, e.g. `Meta+R\tAlt+F2`.

Use the `kwriteconfig-edit` skill for safe edits:

```bash
kwriteconfig6 --file kglobalshortcutsrc --group "krunner.desktop" \
  --key "_launch" "Alt+Space,Alt+Space,KRunner"
```

Then reload:

```bash
qdbus org.kde.kglobalaccel /kglobalaccel org.kde.KGlobalAccel.reloadConfiguration
# or restart the daemon
kquitapp6 kglobalaccel && kstart kglobalaccel
```

## Custom command shortcuts (khotkeys)

Khotkeys / `kglobalaccel` custom shortcuts live in `~/.config/khotkeysrc`. Editing this file directly is fragile — prefer the System Settings UI for new entries, then version-control the resulting file. Applying changes:

```bash
qdbus org.kde.kded6 /modules/khotkeys reread_configuration
```

## Notes

- On Wayland, Plasma owns the global shortcut keymap; X11-only tools (`xbindkeys`) won't work.
- For per-application accelerators, see `~/.config/kdeglobals` `[Shortcuts]` and per-app `*shortcutsrc` files.
