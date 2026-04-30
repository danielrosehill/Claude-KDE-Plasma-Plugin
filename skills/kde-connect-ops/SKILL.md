---
name: kde-connect-ops
description: Pair, list, ping, find, and send files / SMS / clipboard via kdeconnect-cli to phones and other KDE Connect devices. Use for scripted phone↔desktop interactions.
---

# KDE Connect Operations

CLI: `kdeconnect-cli`. Daemon: `kdeconnectd` (autostarts in Plasma).

## Discovery

```bash
kdeconnect-cli --list-devices                # all known devices
kdeconnect-cli --list-devices --id-only      # paired and reachable, IDs only
kdeconnect-cli --refresh                     # re-scan the LAN
```

## Pairing

```bash
kdeconnect-cli --pair --device <device-id>
kdeconnect-cli --unpair --device <device-id>
```

(Confirm on the other device.)

## Ping / find

```bash
kdeconnect-cli --device <device-id> --ping
kdeconnect-cli --device <device-id> --ping-msg "Hello from claude"
kdeconnect-cli --device <device-id> --ring        # find phone — rings it
```

## Send a file / URL

```bash
kdeconnect-cli --device <device-id> --share /path/to/file.pdf
kdeconnect-cli --device <device-id> --share-text "https://example.com"
```

## SMS (Android with KDE Connect SMS plugin)

```bash
kdeconnect-cli --device <device-id> --send-sms "Hi" --destination +972501234567
```

## Clipboard sync

Triggered automatically when both ends have the clipboard plugin enabled — verify in `kdeconnect-cli --device <id> --list-plugins`.

## Notes

- LAN discovery requires UDP 1716 and TCP 1716–1764 open on both ends.
- On a restrictive firewall, allow `kdeconnectd` explicitly.
