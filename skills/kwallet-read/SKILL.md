---
name: kwallet-read
description: Read a single password / map / binary entry from KWallet by wallet name, folder, and key. Triggers a user consent prompt the first time the calling app id is seen.
---

# KWallet Read

## Open

```bash
WALLET="kdewallet"
APPID="claude-kde-plasma"
HANDLE=$(qdbus org.kde.kwalletd6 /modules/kwalletd6 open "$WALLET" 0 "$APPID")
[ "$HANDLE" = "-1" ] && { echo "User declined to unlock"; exit 1; }
```

## Read a password entry

```bash
FOLDER="Passwords"
KEY="github-token"
qdbus org.kde.kwalletd6 /modules/kwalletd6 readPassword "$HANDLE" "$FOLDER" "$KEY" "$APPID"
```

## Read a map entry (key→value dict)

```bash
qdbus org.kde.kwalletd6 /modules/kwalletd6 readMap "$HANDLE" "$FOLDER" "$KEY" "$APPID"
```

## Read a binary entry

```bash
qdbus org.kde.kwalletd6 /modules/kwalletd6 readEntry "$HANDLE" "$FOLDER" "$KEY" "$APPID"
```

## Determine entry type

```bash
qdbus org.kde.kwalletd6 /modules/kwalletd6 entryType "$HANDLE" "$FOLDER" "$KEY" "$APPID"
# 0=unknown 1=password 2=stream/binary 3=map
```

## Close

```bash
qdbus org.kde.kwalletd6 /modules/kwalletd6 close "$HANDLE" false "$APPID"
```

## Security notes

- Don't echo the value into a shell history-visible context. Pipe directly to the consumer or capture into a `read -s` variable.
- The first call from a new `$APPID` triggers a one-shot user prompt; subsequent calls are silent until session end.
- For a CLI alternative, `kwallet-query` exists but is less scriptable: `kwallet-query -f Passwords -r github-token kdewallet`.

> **Stub skill** — single-entry read is wired. Bulk export, JSON output, and integration with `op-vault` for migration paths are TODO.
