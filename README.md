# KDE-Plasma Plugin

![Status: WIP](https://img.shields.io/badge/status-work%20in%20progress-orange)
![Stage: Placeholder](https://img.shields.io/badge/stage-placeholder-lightgrey)
![Plasma 6](https://img.shields.io/badge/Plasma-6%20(Wayland)-1d99f3)
![License: MIT](https://img.shields.io/badge/license-MIT-green)

> **Work in progress — treat this as a placeholder.**
> The skills here are a first pass at mapping the surface, not a finished toolkit.
> Several are unverified against a live Plasma 6 Wayland session, and at least one
> (`keyboard-shortcuts`) documents an approach that is known not to work — see
> [Known-broken](#known-broken). Not yet published to the plugin marketplace.

Claude Code plugin for **runtime operation of a KDE Plasma (Wayland) desktop**.

## The idea: distro plugin × desktop-environment plugin

A Linux desktop is two layers, and they change independently. Package management,
services, journald, filesystem and hardware are properties of the **distribution**.
Panels, widgets, window management, global shortcuts, theming, clipboard and file
indexing are properties of the **desktop environment**.

Most desktop-management tooling smears the two together and ends up either
distro-specific and DE-blind, or generic and unable to touch anything you can see.
This plugin takes the other route: it covers **only** the Plasma half, so it can be
paired with whichever distro plugin matches the machine.

```
  distro layer          ×          DE layer          =   the machine
  ────────────────                 ────────              ───────────
  desktop-manager                  kde-plasma           Ubuntu + Plasma 6
  linux-debugging                  (this plugin)
  linux-system-optimisation        …a gnome-shell        Fedora + GNOME
                                     equivalent
```

Companion plugins in the same marketplace that sit on the distro side:

| Plugin | Layer | Covers |
| --- | --- | --- |
| `desktop-manager` | distro | Machine profiling, packages, config application, hardware, services |
| `linux-debugging` | distro | journald, boot logs, kdump, crash/freeze analysis |
| `linux-system-optimisation` | distro | CPU/GPU/disk benchmarks, governors, sysctl, disk usage |
| `kde-plasmoid-dev` | DE (build-time) | Writing Plasma widgets — QML scaffolding, packaging, 5→6 migration |
| **`kde-plasma`** | **DE (run-time)** | **This plugin — operating the Plasma desktop you already have** |

The split against `kde-plasmoid-dev` is build-time versus run-time: that plugin
*makes* widgets, this one *drives* the desktop.

## Skills

Present today. Maturity is honest, not aspirational.

| Skill | What it does | State |
| --- | --- | --- |
| `kwin-scripting` | List / install / load / remove KWin scripts via `kpackagetool6` + `org.kde.KWin` D-Bus | Unverified |
| `plasmoid-manage` | List, add, remove, configure Plasma widgets on panels and desktop | Unverified |
| `panel-layout-backup` | Back up / restore panel layouts (`plasma-org.kde.plasma.desktop-appletsrc` + companions) | Unverified |
| `virtual-desktops-activities` | Create, rename, switch, remove virtual desktops and KActivities | Unverified |
| `keyboard-shortcuts` | Inspect and edit KGlobalAccel global shortcuts | **Partly broken** |
| `theme-switch` | Look-and-feel packages, colour schemes, icon/cursor themes, GTK theming via `plasma-apply-*` | Unverified |
| `screen-config-save` / `screen-config-apply` | Save and apply multi-monitor layouts via `kscreen-doctor` | Unverified |
| `kde-connect-ops` | Pair, list, ping, find-device, send files/SMS via `kdeconnect-cli` | Unverified |
| `klipper-ops` | Read, clear, push Klipper clipboard history via `qdbus org.kde.klipper` | Unverified |
| `baloo-ops` | Inspect, pause, resume, reset the Baloo file indexer | Unverified |
| `kwallet-status` / `-list` / `-read` / `-write` | KWallet inspection and entry management | Unverified |
| `kwriteconfig-edit` | Backup-before-write wrapper for KDE config files | Unverified |
| `plasma-restart` | Restart Plasma shell / KWin / component daemons | Unverified |
| `qdbus-introspect` | Discover KDE D-Bus services, objects, methods | Unverified |

"Unverified" means the commands are drawn from KDE documentation and D-Bus
introspection but have not been exercised end-to-end on a live session. Verifying
them is the main body of remaining work.

## Roadmap — target operations

Plasma's operable surface is very large; this is the shortlist worth targeting
next, chosen for things a user actually asks an agent to do mid-session.

### Hardware and output controls

- **Screen brightness** — internal panel and external monitors (DDC/CI). Note the
  standing gap: `kscreen-doctor` does not do backlight, so this needs
  `brightnessctl` / `ddcutil` / the PowerDevil D-Bus interface, and the right one
  depends on the display. See the [`Candela`](https://github.com/danielrosehill/Candela)
  repo for already-mapped Plasma brightness interfaces.
- **Volume and mute** — per-sink and per-application, via PipeWire/`wpctl` and the
  Plasma volume OSD. Includes mic mute, which is the one people actually want bound.
- **Night colour / colour temperature** — toggle and schedule.
- **Power profiles** — performance / balanced / power-saver via PowerDevil.

### Window management

- Query the window list with geometry, desktop, activity, and app id.
- Move, resize, tile, maximise, fullscreen, and pin windows to a desktop or activity.
- Focus and raise a named window; cycle within an application.
- Read and set window rules (`kwinrulesrc`) — the durable form of the above.
- Tiling layout control on Plasma 6's built-in tiling.

### Session and shell

- Lock, log out, suspend, hibernate cleanly (respecting inhibitors).
- Query and set inhibitors — "don't sleep while this runs".
- Notification send / query / do-not-disturb toggle.
- Panel and widget state that survives a Plasma restart.

### Input

- Touchpad and mouse settings — natural scroll, tap-to-click, acceleration.
- Keyboard layout switching and per-layout shortcuts.
- Sort out global-shortcut writes properly (see below) — this gates a lot.

## Known-broken

`keyboard-shortcuts` currently documents two approaches that do not work on
Plasma 6 Wayland, verified on a ThinkPad E15 Gen 3 running Plasma 6:

1. **Writing `~/.config/kglobalshortcutsrc` and reloading via D-Bus.** The write
   lands in the file, but `kglobalaccel` does not reliably pick it up; in practice
   the binding only takes effect after a full logout. Corroborated by
   [KDE Discuss #17995](https://discuss.kde.org/t/plasma-6-method-to-refresh-kglobalaccel-shortcuts/17995).
2. **The khotkeys route.** The `khotkeys` KDED module does not exist in Plasma 6 —
   `qdbus org.kde.kded6 /modules/khotkeys reread_configuration` has nothing to talk
   to. Custom command shortcuts moved into the Shortcuts KCM.

For custom command shortcuts on Plasma 6 Wayland, **System Settings → Shortcuts →
Add New → Command or Script** is currently the only reliable path. Full write-up,
including what else was tried, in
[`custom-keybindings`](https://github.com/danielrosehill/custom-keybindings).

Finding a scriptable route that actually applies is an open problem and a
prerequisite for most of the Input roadmap above.

## Requirements

- KDE Plasma 6 on Wayland (primary target).
- Plasma 5 fallbacks (`kwriteconfig5`, `kquitapp5`) are referenced where the KF5
  tool names differ, but Plasma 5 is not tested.
- `qdbus6`, `kpackagetool6`, `kscreen-doctor`, `kwriteconfig6` — normally present
  with a standard Plasma install.

## Installation

Not in the marketplace yet. Install directly from this repository:

```bash
claude plugins install danielrosehill/Claude-KDE-Plasma-Plugin
```

Or clone and point Claude Code at the working copy while iterating:

```bash
git clone https://github.com/danielrosehill/Claude-KDE-Plasma-Plugin.git
claude plugins install ./Claude-KDE-Plasma-Plugin --scope project
```

## Scope

Runtime *operation* of an installed Plasma desktop. Explicitly out of scope:

- Building Plasma widgets → `kde-plasmoid-dev`.
- Anything below the DE — packages, services, kernel, filesystem → the distro-layer
  plugins listed above.
- GUI automation (clicking things, screenshots, accessibility tree). If you need
  that, [`kwin-mcp`](https://github.com/isac322/kwin-mcp) is an MCP server that
  drives a Plasma 6 Wayland session via KWin's EIS interface. It is complementary,
  not overlapping: it operates the desktop *through the UI*, this plugin operates it
  *through configuration and D-Bus*.

## License

MIT — see [LICENSE](LICENSE).
