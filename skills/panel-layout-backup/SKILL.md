---
name: panel-layout-backup
description: Back up and restore Plasma panel and desktop layout — the configuration that defines panels, applets on those panels, screen edges, and widget positions. Use before risky theme/layout changes or to clone setup across machines.
---

# Panel Layout Backup & Restore

Plasma's panel/desktop layout lives primarily in:

- `~/.config/plasma-org.kde.plasma.desktop-appletsrc` — panels, containments, applets, their positions and config.
- `~/.config/plasmashellrc` — plasmashell preferences.
- `~/.config/plasmarc` — global Plasma settings.
- `~/.config/kwinrc` — window manager config (often paired with layout changes).

## Backup

Snapshot to the plugin data dir, timestamped:

```bash
DATA="${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/kde-plasma/data/panel-layouts"
mkdir -p "$DATA"
TS=$(date +%Y%m%d-%H%M%S)
DEST="$DATA/$TS"
mkdir -p "$DEST"
for f in plasma-org.kde.plasma.desktop-appletsrc plasmashellrc plasmarc kwinrc kglobalshortcutsrc; do
  [ -f "$HOME/.config/$f" ] && cp -a "$HOME/.config/$f" "$DEST/"
done
echo "Backed up to $DEST"
```

## Restore

```bash
# Stop plasmashell first so it doesn't overwrite on exit
kquitapp6 plasmashell
sleep 1
cp -a "$DEST"/*rc "$HOME/.config/"
kstart plasmashell
```

## Notes

- Plasma writes panel state on shutdown — restoring while `plasmashell` is running will be silently undone.
- For full theming + layout, also snapshot `~/.config/kdeglobals` and `~/.local/share/plasma/`.
- To diff two snapshots: `diff -ru <old>/ <new>/`.
