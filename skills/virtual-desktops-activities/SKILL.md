---
name: virtual-desktops-activities
description: Manage Plasma virtual desktops and KActivities — list, create, rename, switch, remove. Use when the user wants to script desktop layout, set up per-task activities, or rebuild a clobbered desktop config.
---

# Virtual Desktops & Activities

Plasma has two orthogonal grouping mechanisms — virtual desktops (lightweight, per-window) and activities (heavyweight, per-context with persistent state).

## Virtual desktops

D-Bus interface: `org.kde.KWin /VirtualDesktopManager`.

```bash
# List
qdbus org.kde.KWin /VirtualDesktopManager desktopIds
qdbus org.kde.KWin /VirtualDesktopManager desktopName <id>

# Count
qdbus org.kde.KWin /VirtualDesktopManager count

# Create / remove
qdbus org.kde.KWin /VirtualDesktopManager createDesktop <position> "<name>"
qdbus org.kde.KWin /VirtualDesktopManager removeDesktop <id>

# Switch
qdbus org.kde.KWin /VirtualDesktopManager setCurrent <id>
```

Persistent config: `~/.config/kwinrc` `[Desktops]`.

## Activities

CLI: `kactivitymanagerd` runs the daemon; user-facing CLI is `qdbus org.kde.ActivityManager /ActivityManager/Activities`.

```bash
# List
qdbus org.kde.ActivityManager /ActivityManager/Activities ListActivities
qdbus org.kde.ActivityManager /ActivityManager/Activities ActivityName <uuid>

# Current
qdbus org.kde.ActivityManager /ActivityManager/Activities CurrentActivity

# Add / remove
qdbus org.kde.ActivityManager /ActivityManager/Activities AddActivity "<name>"
qdbus org.kde.ActivityManager /ActivityManager/Activities RemoveActivity <uuid>

# Switch
qdbus org.kde.ActivityManager /ActivityManager/Activities SetCurrentActivity <uuid>
```

## Notes

- Window-to-desktop / window-to-activity assignment lives in window rules (`~/.config/kwinrulesrc`).
- Activity state (which apps are open per activity) is stored under `~/.local/share/kactivitymanagerd/`.
