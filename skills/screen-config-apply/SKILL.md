---
name: screen-config-apply
description: Apply a saved Plasma screen-configuration preset, or set ad-hoc resolution/position/scale/primary on connected outputs via kscreen-doctor. Use to restore a saved monitor layout or to script display changes.
---

# Apply Screen Configuration

## Apply a saved preset (from `screen-config-save`)

```bash
DATA="${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/kde-plasma/data/screen-presets"
NAME="${1:-default}"

# Restore the kscreen daemon's snapshot dir
if [ -d "$DATA/$NAME.kscreen.d" ]; then
  cp -r "$DATA/$NAME.kscreen.d/." ~/.local/share/kscreen/
  # Nudge the daemon to re-read
  qdbus org.kde.kded6 /modules/kscreen org.kde.kscreen.applyConfig 2>/dev/null \
    || (kquitapp6 kded6 && kstart kded6)
  echo "Applied preset '$NAME'"
else
  echo "No preset '$NAME' at $DATA"; exit 1
fi
```

## Ad-hoc changes via `kscreen-doctor`

```bash
# Inspect what's connected
kscreen-doctor -o

# Set mode (resolution.refresh)
kscreen-doctor output.DP-1.mode.3840x2160@60

# Set position (x,y)
kscreen-doctor output.DP-1.position.0,0
kscreen-doctor output.HDMI-A-1.position.3840,0

# Scale (1.0, 1.25, 1.5, 2.0…)
kscreen-doctor output.DP-1.scale.1.5

# Primary
kscreen-doctor output.DP-1.primary

# Enable / disable
kscreen-doctor output.HDMI-A-1.disable
kscreen-doctor output.HDMI-A-1.enable

# Rotation
kscreen-doctor output.DP-2.rotation.left
```

Multiple changes can be chained on one command line; `kscreen-doctor` applies them atomically.

## Notes

- On Wayland, fractional scale is per-output; on X11 it's session-wide. Plasma 6 / Wayland is the assumed target.
- After applying, verify with `kscreen-doctor -o`.
- If a config doesn't stick across replug, the daemon's auto-snapshot is overriding — re-run apply, then **disconnect-reconnect once** so the daemon writes a new auto-snapshot for that output combo.
