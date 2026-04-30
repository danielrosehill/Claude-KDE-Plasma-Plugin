---
name: kwallet-list
description: List wallets, folders within a wallet, and entries within a folder. Use to enumerate what's stored in KWallet without revealing values.
---

# KWallet List

Read-only enumeration. Does not reveal entry values — use `kwallet-read` for that.

## Wallets

```bash
qdbus org.kde.kwalletd6 /modules/kwalletd6 wallets
```

## Open a wallet (required before listing folders/entries)

```bash
WALLET="kdewallet"
HANDLE=$(qdbus org.kde.kwalletd6 /modules/kwalletd6 open "$WALLET" 0 "claude-kde-plasma")
echo "$HANDLE"
```

A handle of `-1` means the user cancelled the unlock prompt.

## Folders

```bash
qdbus org.kde.kwalletd6 /modules/kwalletd6 folderList "$HANDLE" "claude-kde-plasma"
```

## Entries in a folder

```bash
FOLDER="Passwords"
qdbus org.kde.kwalletd6 /modules/kwalletd6 entryList "$HANDLE" "$FOLDER" "claude-kde-plasma"
```

## Close

```bash
qdbus org.kde.kwalletd6 /modules/kwalletd6 close "$HANDLE" false "claude-kde-plasma"
```

## Notes

- The trailing `"claude-kde-plasma"` is the **app id** — KWallet uses it for access control. Reuse the same id across this plugin's skills so the user only sees one consent prompt.
- Common folders: `Passwords`, `Form Data`, `Network Management`, `Chromium Keys`.

> **Stub skill** — listing implemented. Search/filter, output formats, and bulk export need to be built out.
