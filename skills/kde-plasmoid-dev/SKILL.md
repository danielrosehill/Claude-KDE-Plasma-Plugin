---
name: kde-plasmoid-dev
description: Use when the user wants to develop, scaffold, debug, package, or install a KDE Plasma 6 plasmoid (desktop or panel widget). Triggers on phrases like "build a plasmoid", "KDE widget", "Plasma 6 applet", "kpackagetool6", "QML plasmoid", "PlasmoidItem". Plasma 6 / Qt 6 only — does not cover the legacy Plasma 5 stack.
---

# KDE Plasma 6 Plasmoid Development

Plasmoids are QML applications loaded by the Plasma 6 shell. A plasmoid is a directory with a `metadata.json` and a `contents/ui/main.qml`, packaged and installed via `kpackagetool6`.

## Ground in current docs first

Plasma APIs and packaging conventions change between minor releases. **Always fetch the current canonical docs before scaffolding or debugging** — do not rely on the inline summary below as the source of truth. Fetch the relevant pages with WebFetch (or the user's preferred docs tool) and use them as the authority.

Authoritative references (Plasma 6):

- **Widget development index** — https://develop.kde.org/docs/plasma/widget/
- **Getting started / first widget tutorial** — https://develop.kde.org/docs/plasma/widget/tutorial/
- **`metadata.json` properties reference** — https://develop.kde.org/docs/plasma/widget/properties/
- **Configuration (KConfigXT + config.qml)** — https://develop.kde.org/docs/plasma/widget/configuration/
- **Translation / i18n** — https://develop.kde.org/docs/plasma/widget/translations-i18n/
- **Plasma 5 → 6 porting guide** — https://develop.kde.org/docs/plasma/widget/porting-plasma6/
- **Examples (official `plasma-sdk` examples repo)** — https://invent.kde.org/plasma/plasma-sdk
- **Plasma framework API** — https://api.kde.org/frameworks/plasma-framework/html/
- **Kirigami components** — https://develop.kde.org/docs/getting-started/kirigami/
- **Community Developer Guide (note: largely Plasma 5 era — useful for context, not authoritative for P6)** — https://community.kde.org/Plasma/DeveloperGuide

When the user asks for an API, schema field, or import path, fetch the relevant URL above rather than guessing.

## Basic conventions (Plasma 6)

These hold at the time of writing — verify against the docs above for any project that ships.

### Directory layout

```
my-plasmoid/
├── metadata.json
└── contents/
    ├── ui/
    │   ├── main.qml                  # required — root must be PlasmoidItem
    │   ├── CompactRepresentation.qml # optional — panel / collapsed view
    │   └── FullRepresentation.qml    # optional — popup / expanded view
    ├── config/
    │   ├── main.xml                  # KConfigXT schema
    │   └── config.qml                # ConfigModel of pages
    └── icon.svg                      # optional, if not using a theme icon
```

### `metadata.json`

```json
{
  "KPlugin": {
    "Id": "org.example.myplasmoid",
    "Name": "My Plasmoid",
    "Description": "One-line description",
    "Icon": "preferences-desktop-plasma",
    "Authors": [{ "Name": "Author", "Email": "author@example.com" }],
    "Category": "Utilities",
    "License": "MIT",
    "Version": "1.0",
    "Website": "https://example.com"
  },
  "KPackageStructure": "Plasma/Applet",
  "X-Plasma-API-Minimum-Version": "6.0"
}
```

The `Id` must be reverse-DNS, unique, and **must match the directory name** when installed.

### `main.qml` skeleton

```qml
import QtQuick
import QtQuick.Layouts
import org.kde.plasma.plasmoid
import org.kde.plasma.components as PlasmaComponents
import org.kde.kirigami as Kirigami

PlasmoidItem {
    id: root

    compactRepresentation: PlasmaComponents.Label { text: "MP" }

    fullRepresentation: ColumnLayout {
        Layout.minimumWidth: Kirigami.Units.gridUnit * 12
        Layout.minimumHeight: Kirigami.Units.gridUnit * 8

        PlasmaComponents.Label {
            Layout.alignment: Qt.AlignCenter
            text: "Hello from My Plasmoid"
        }
    }
}
```

The root **must** be `PlasmoidItem` (from `org.kde.plasma.plasmoid`). Imports drop the version suffix in Plasma 6 (`org.kde.plasma.components`, not `... 3.0`).

### Install / test cycle

```bash
# Install or upgrade to user scope (~/.local/share/plasma/plasmoids/<id>/)
kpackagetool6 -t Plasma/Applet -i ./my-plasmoid       # first time
kpackagetool6 -t Plasma/Applet -u ./my-plasmoid       # upgrade in place
kpackagetool6 -t Plasma/Applet -r org.example.myplasmoid

# Live preview without touching the panel
plasmoidviewer6 -a ./my-plasmoid
# or, for an installed plasmoid:
plasmawindowed org.example.myplasmoid

# Reload plasmashell after an upgrade
kquitapp6 plasmashell && kstart plasmashell
```

Edits to QML files inside `~/.local/share/plasma/plasmoids/<id>/contents/ui/` take effect after a plasmashell reload (or remove + re-add the widget on the panel).

### Debugging

- Tail QML errors and `console.log()`: `journalctl --user -f -t plasmashell`
- Verbose QML logging: `kquitapp6 plasmashell; QT_LOGGING_RULES="qml=true" plasmashell --replace`
- `plasmoidviewer6` runs the plasmoid in isolation — the fastest dev loop, and prints errors to the terminal directly.

### Common gotchas

- **Wrong root type** — bare `Item` instead of `PlasmoidItem` silently fails to expose `compactRepresentation` / `fullRepresentation`.
- **Versioned imports** — Plasma 5 imports like `org.kde.plasma.components 3.0` fail in Plasma 6; drop the version.
- **`Id` mismatch** — directory name and `KPlugin.Id` must agree.
- **Trailing commas in `metadata.json`** — JSON is strict; `kpackagetool6` will refuse to install.
- **Icon resolution** — `KPlugin.Icon` must be a freedesktop icon name (e.g. `preferences-desktop-plasma`) or omitted if shipping `contents/icon.svg`.

## Output convention when scaffolding

When the user asks to scaffold a new plasmoid:

1. Confirm the running stack: `plasmashell --version` (must be 6.x), `kpackagetool6 --version`. If they're on Plasma 5, stop and tell the user this skill is Plasma 6 only.
2. Ask for: plugin id (reverse-DNS), display name, one-line description, where to put the source (suggest `~/repos/` — never `~/.claude/`).
3. Fetch https://develop.kde.org/docs/plasma/widget/properties/ to confirm the current `metadata.json` schema before writing the file.
4. Write `metadata.json` and `contents/ui/main.qml`. Add `contents/config/` only if the user wants settings.
5. Install with `kpackagetool6 -t Plasma/Applet -i ./<dir>` and tell the user how to add it to their panel.
6. Offer to tail `journalctl --user -f -t plasmashell` in a background shell during iteration.
