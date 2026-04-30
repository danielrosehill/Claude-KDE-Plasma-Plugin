---
name: klipper-ops
description: Read, clear, and push entries to the Klipper clipboard history via D-Bus. Use when the user wants to inspect what's been copied recently, clear sensitive history, or programmatically stage clipboard content.
---

# Klipper Operations

Klipper is Plasma's clipboard manager. D-Bus interface: `org.kde.klipper /klipper`.

## Read

```bash
# Current clipboard contents
qdbus org.kde.klipper /klipper getClipboardContents

# History (newest first)
qdbus org.kde.klipper /klipper getClipboardHistoryMenu

# Specific item by index (0 = most recent)
qdbus org.kde.klipper /klipper getClipboardHistoryItem 0
```

## Write

```bash
qdbus org.kde.klipper /klipper setClipboardContents "text to push"
```

## Clear history

```bash
qdbus org.kde.klipper /klipper clearClipboardHistory
qdbus org.kde.klipper /klipper clearClipboardContents
```

## Notes

- Klipper config: `~/.config/klipperrc` — `MaxClipItems`, `IgnoreSelection`, `KeepClipboardContents`.
- For sensitive workflows, add app patterns to the ignore list in `klipperrc` `[General] IgnoreSelection=true` plus per-app rules.
- On Wayland, clipboard access requires the source app's window to have been focused at copy time; some apps (password managers) deliberately bypass Klipper history.
