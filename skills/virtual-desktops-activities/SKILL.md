---
name: virtual-desktops-activities
description: Manage Plasma virtual desktops and KActivities — list, create, rename, switch, remove. Use when the user wants to script desktop layout, set up per-task activities, or rebuild a clobbered desktop config.
---

# Virtual Desktops & Activities

Plasma has two orthogonal grouping mechanisms — virtual desktops (lightweight, per-window) and activities (heavyweight, per-context with persistent state).

## Virtual desktops

D-Bus interface: `org.kde.KWin` `/VirtualDesktopManager`, `org.kde.KWin.VirtualDesktopManager`.

`desktops`, `current`, `count`, `rows` and `navigationWrappingAround` are **properties**,
not methods. There is no `desktopIds`, `desktopName` or `setCurrent` method. Read them
with `org.freedesktop.DBus.Properties`. Verified on Plasma 6.6.6 Wayland, 2026-10-19.

```bash
VDM="org.kde.KWin /VirtualDesktopManager"
P=org.freedesktop.DBus.Properties
I=org.kde.KWin.VirtualDesktopManager

# List — a(uss): (0-based position, id, name)
qdbus6 --literal $VDM $P.Get $I desktops

# Count / current id
qdbus6 $VDM $P.Get $I count
qdbus6 $VDM $P.Get $I current

# Create / rename / remove (these are real methods)
qdbus6 $VDM createDesktop <position> "<name>"
qdbus6 $VDM setDesktopName <id> "<name>"
qdbus6 $VDM removeDesktop <id>

# Switch
qdbus6 $VDM $P.Set $I current <id>
```

Persistent config: `~/.config/kwinrc` `[Desktops]` (`Id_N`, `Name_N`, `Number`, `Rows`).
`Name_N` is absent for desktops still on the default "Desktop N" name.

Per-output desktops (independent desktop per monitor) arrived in Plasma 6.7. They are
opt-in: `kwinrc` `[Windows]` `PerOutputVirtualDesktops=true`. On 6.7+ the `current`
property reports the desktop of the *active* output. To make Claude work on a dedicated
desktop/monitor, see the
[`kde-agent-workspace`](https://github.com/danielrosehill/Claude-KDE-Agent-Workspace-Plugin) plugin.

## Activities

CLI: `kactivitymanagerd` runs the daemon; user-facing CLI is `qdbus org.kde.ActivityManager /ActivityManager/Activities`.
The method names below match the live interface (introspected 2026-10-19, Plasma 6.6.6).
`ListActivitiesWithInformation` returns `a(ssssi)` in a single call.

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
