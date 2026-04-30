# KDE-Plasma Plugin

Claude Code plugin for runtime management of a KDE Plasma (Wayland) desktop. Pairs alongside generic Linux desktop-management plugins by bundling Plasma-specific things: KWin scripting, plasmoids, panels, virtual desktops & activities, KGlobalAccel shortcuts, look-and-feel/theme switching, KDE Connect, Klipper, Baloo, `kwriteconfig`, `qdbus` introspection, and Plasma component restart helpers.

Targets Plasma 6 (Wayland) primarily; falls back to Plasma 5 tooling (`kwriteconfig5`, `kquitapp5`) where the user is still on KF5.

## Skills

- `kwin-scripting` — list / install / load / remove KWin scripts via `kpackagetool6` and `org.kde.KWin` D-Bus.
- `plasmoid-manage` — list, add, remove, and configure Plasma widgets on panels and the desktop.
- `virtual-desktops-activities` — manage Plasma virtual desktops and KActivities (create, rename, switch, remove).
- `keyboard-shortcuts` — inspect and edit KGlobalAccel shortcuts via `kglobalaccel*` and `kwriteconfig`.
- `panel-layout-backup` — back up and restore Plasma panel layouts (`plasma-org.kde.plasma.desktop-appletsrc` + companions).
- `theme-switch` — apply look-and-feel packages, color schemes, icon themes, cursor themes, and GTK theming via `plasma-apply-*` / `lookandfeeltool`.
- `kde-connect-ops` — pair, list, ping, find-device, and send files / SMS via `kdeconnect-cli`.
- `klipper-ops` — read, clear, and push the Klipper clipboard history via `qdbus org.kde.klipper`.
- `baloo-ops` — inspect, pause, resume, and reset the Baloo file indexer; check disabled paths.
- `kwriteconfig-edit` — safe wrapper for `kwriteconfig6` / `kwriteconfig5` edits to KDE config files (with backup-before-write).
- `plasma-restart` — restart `plasmashell`, `krunner`, `kglobalaccel`, `kwin_wayland` (when safe) without logging out.
- `qdbus-introspect` — list KDE D-Bus services, interfaces, and methods to discover automation surfaces.
- `screen-config-save` — snapshot the current multi-monitor layout (mode, position, scale, primary, rotation) to a named preset.
- `screen-config-apply` — restore a saved screen preset, or set ad-hoc display state via `kscreen-doctor`.
- `kwallet-status` — check whether kwalletd is running, enabled, and which wallets exist *(stub)*.
- `kwallet-list` — list wallets, folders, and entries *(stub)*.
- `kwallet-read` — read a single password / map / binary entry *(stub)*.
- `kwallet-write` — write or update a wallet entry *(stub)*.

## Installation

```bash
claude plugins marketplace add danielrosehill/Claude-Code-Plugins
claude plugins install kde-plasma@danielrosehill
```

Or, ad-hoc per-project:

```bash
claude plugins install kde-plasma@danielrosehill --scope project
```

## Scope

Runtime *operation* of an installed Plasma desktop. For developing new Plasma widgets (QML scaffolding, plasmoid packaging, Plasma 5→6 migration), use the companion `kde-plasmoid-dev` plugin instead.

## License

MIT — see [LICENSE](LICENSE).
