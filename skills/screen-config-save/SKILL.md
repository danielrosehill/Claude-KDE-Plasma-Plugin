---
name: screen-config-save
description: Save the current Plasma multi-monitor screen configuration (resolutions, positions, scale, refresh rate, primary, rotation) to a named preset. Use before plugging/unplugging displays or when you've just got a layout dialled in and want to preserve it.
---

# Save Screen Configuration

Plasma's `KScreen` daemon manages monitor layout. Two surfaces are useful:

- **`kscreen-doctor`** — read/write live state.
- **`~/.local/share/kscreen/`** — daemon's per-output-set persistence (one JSON file per detected output combination, named by a hash of connected outputs).

## Capture live state

```bash
DATA="${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/kde-plasma/data/screen-presets"
mkdir -p "$DATA"
NAME="${1:-default}"

# Human-readable snapshot
kscreen-doctor -o > "$DATA/$NAME.txt"

# Machine-readable JSON via kscreen daemon
cp -r ~/.local/share/kscreen "$DATA/$NAME.kscreen.d"

echo "Saved screen preset '$NAME' to $DATA"
```

## What gets captured

`kscreen-doctor -o` lists per-output:

- ID and name (e.g. `DP-1`, `HDMI-A-2`).
- Enabled state, primary flag.
- Mode (resolution + refresh rate).
- Position (x,y).
- Scale factor.
- Rotation (`none|left|right|inverted`).
- Replication source (mirror).

## Notes

- Save before:
  - Plugging in a dock or external display you don't always use.
  - Major Plasma updates (KScreen schema occasionally changes).
  - Any session where you've manually tuned scale/refresh and want a known-good fallback.
- Pair with `screen-config-apply` to restore.
- The Plasma daemon writes its own snapshots to `~/.local/share/kscreen/` automatically — but those are keyed by the connected-output hash, so unplugging an output before saving means you can't recover that exact layout via the daemon alone. The plugin preset is independent of that.
