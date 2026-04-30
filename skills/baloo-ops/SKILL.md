---
name: baloo-ops
description: Inspect, pause, resume, and reset the Baloo file indexer. Use when search is missing files, indexing is hammering disk/CPU, or the user wants to add/remove indexed paths.
---

# Baloo File Indexer Operations

Baloo backs KRunner file search and Dolphin's content search. Daemon: `baloo_file`. CLI: `balooctl6` (Plasma 6) / `balooctl` (Plasma 5).

## Status

```bash
balooctl6 status
balooctl6 monitor          # live progress
balooctl6 indexSize        # disk usage of the index
```

## Pause / resume / disable

```bash
balooctl6 suspend
balooctl6 resume
balooctl6 disable          # stop daemon and disable on boot
balooctl6 enable
```

## Reset

```bash
balooctl6 purge            # wipe and rebuild
# Or nuke the on-disk index manually:
balooctl6 disable
rm -rf ~/.local/share/baloo
balooctl6 enable
```

## Configure included / excluded paths

Config: `~/.config/baloofilerc`.

```ini
[General]
exclude folders[$e]=$HOME/Downloads,$HOME/.cache,$HOME/VirtualBox VMs
only basic indexing=false
```

Apply:

```bash
balooctl6 disable && balooctl6 enable
```

## Check a file

```bash
balooctl6 indexed /path/to/file       # is it indexed?
balooctl6 fileinfo /path/to/file      # extracted metadata
```

## Notes

- Default index location: `~/.local/share/baloo/index`.
- High CPU/IO? `balooctl6 suspend`, then exclude noisy directories before resuming.
- Baloo is **content-aware** for many MIME types — including PDFs and Office docs — but not arbitrary binary blobs.
