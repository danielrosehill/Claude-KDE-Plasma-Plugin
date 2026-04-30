---
name: qdbus-introspect
description: Discover KDE D-Bus services, objects, interfaces, and methods. Use when scripting against an unfamiliar Plasma component, or hunting for the right call to automate a task.
---

# qdbus Introspection

Plasma exposes most of its scriptable surface over D-Bus. `qdbus6` (Plasma 6) / `qdbus` (Plasma 5) is the discovery tool.

## Tooling

```bash
QD=$(command -v qdbus6 || command -v qdbus-qt6 || command -v qdbus)
```

`busctl --user` is a more powerful systemd alternative; useful when `qdbus` output is ambiguous.

## List services

```bash
$QD                           # all services on the session bus
$QD | grep -i kde
```

Common services:

- `org.kde.KWin` — window manager and scripting.
- `org.kde.plasmashell` — panels, widgets, scripting.
- `org.kde.kglobalaccel` — global shortcuts.
- `org.kde.klipper` — clipboard.
- `org.kde.ActivityManager` — activities.
- `org.kde.kded6` — background module host.
- `org.kde.dolphin` / `org.kde.konsole-N` — running app instances.

## List objects within a service

```bash
$QD org.kde.KWin
```

## List methods/properties of an object

```bash
$QD org.kde.KWin /KWin
$QD org.kde.plasmashell /PlasmaShell
```

## Call a method

```bash
$QD <service> <object> <method> [args...]
# e.g.
$QD org.kde.KWin /KWin reconfigure
$QD org.kde.plasmashell /PlasmaShell evaluateScript 'print(panels().length)'
```

## Watch signals

```bash
dbus-monitor --session "interface='org.kde.KWin'"
busctl --user monitor org.kde.KWin
```

## Tips

- Use `busctl --user introspect <service> <object>` for a typed signature view.
- Plasma scripting reference (richer than D-Bus alone): https://develop.kde.org/docs/plasma/scripting/
- KWin scripting reference: https://develop.kde.org/docs/plasma/kwin/api/
