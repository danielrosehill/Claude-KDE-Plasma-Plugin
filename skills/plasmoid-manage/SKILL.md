---
name: plasmoid-manage
description: List, install, remove, and add Plasma widgets (plasmoids) to panels and the desktop. Use when the user wants to manage Plasma widgets at runtime — not develop new ones.
---

# Plasmoid Management

Runtime management of Plasma widgets. For *developing* new plasmoids, use the `kde-plasmoid-dev` plugin instead.

## Tooling

- `kpackagetool6 --type=Plasma/Applet` (Plasma 6) or `kpackagetool5 --type=Plasma/Applet` (Plasma 5).
- `plasmashell` evaluation via `qdbus org.kde.plasmashell /PlasmaShell evaluateScript "<JS>"`.

## Common operations

### List installed plasmoids

```bash
kpackagetool6 --type=Plasma/Applet --list
```

### Install from .plasmoid file or directory

```bash
kpackagetool6 --type=Plasma/Applet --install /path/to/foo.plasmoid
# Update an existing one:
kpackagetool6 --type=Plasma/Applet --upgrade /path/to/foo.plasmoid
```

### Remove

```bash
kpackagetool6 --type=Plasma/Applet --remove org.kde.plasma.foo
```

### Add a widget to a panel (scripted)

```bash
qdbus org.kde.plasmashell /PlasmaShell evaluateScript '
  var panel = panels()[0];
  panel.addWidget("org.kde.plasma.foo");
'
```

### Add a widget to the desktop

```bash
qdbus org.kde.plasmashell /PlasmaShell evaluateScript '
  var d = desktops()[0];
  d.addWidget("org.kde.plasma.foo");
'
```

## Reload after changes

```bash
kquitapp6 plasmashell && kstart plasmashell
```

## Notes

- Plasma scripting API: https://develop.kde.org/docs/plasma/scripting/
- Widget IDs follow reverse-DNS: `org.kde.plasma.<name>`.
