---
name: kwallet-write
description: Write or update a password / map / binary entry in KWallet. Creates the folder if it doesn't exist. Use to seed credentials for KDE apps that don't have their own input flow.
---

# KWallet Write

## Open

```bash
WALLET="kdewallet"
APPID="claude-kde-plasma"
HANDLE=$(qdbus org.kde.kwalletd6 /modules/kwalletd6 open "$WALLET" 0 "$APPID")
[ "$HANDLE" = "-1" ] && { echo "User declined to unlock"; exit 1; }
```

## Ensure folder exists

```bash
FOLDER="Passwords"
qdbus org.kde.kwalletd6 /modules/kwalletd6 hasFolder "$HANDLE" "$FOLDER" "$APPID" \
  | grep -q true || qdbus org.kde.kwalletd6 /modules/kwalletd6 createFolder "$HANDLE" "$FOLDER" "$APPID"
```

## Write a password

```bash
KEY="github-token"
VALUE="ghp_..."
qdbus org.kde.kwalletd6 /modules/kwalletd6 writePassword "$HANDLE" "$FOLDER" "$KEY" "$VALUE" "$APPID"
```

## Write a map

```bash
# Maps are serialised QMap<QString,QString> — easiest to write via kwallet-query
echo -e "user=foo\npass=bar" | kwallet-query -f "$FOLDER" -w "$KEY" "$WALLET"
```

## Delete an entry

```bash
qdbus org.kde.kwalletd6 /modules/kwalletd6 removeEntry "$HANDLE" "$FOLDER" "$KEY" "$APPID"
```

## Close

```bash
qdbus org.kde.kwalletd6 /modules/kwalletd6 close "$HANDLE" false "$APPID"
```

## Notes

- Don't put secrets in command-line arguments on shared systems — they show up in `/proc/<pid>/cmdline`. Read into a variable first via `read -s`, then pass.
- Daniel uses 1Password as the source of truth; KWallet is mainly populated by KDE apps themselves. This skill is for the rare case where a KDE app expects a credential to already exist.

> **Stub skill** — basic write is wired. Idempotent updates, batch import from 1Password / file, and dry-run are TODO.
