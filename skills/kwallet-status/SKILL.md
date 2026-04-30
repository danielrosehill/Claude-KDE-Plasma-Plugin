---
name: kwallet-status
description: Check whether kwalletd is running, which wallets exist, whether the default wallet is open, and whether KWallet is enabled at all. Use as the first step before any other kwallet skill.
---

# KWallet Status

KWallet is Plasma's encrypted credential store. Daemon: `kwalletd6` (Plasma 6) / `kwalletd5` (Plasma 5).

## Is KWallet enabled?

```bash
kreadconfig6 --file kwalletrc --group Wallet --key Enabled --default true
```

If `false`, KWallet is disabled and no operations will work — apps fall back to plain config storage.

## Is the daemon running?

```bash
pgrep -af kwalletd6 || pgrep -af kwalletd5
qdbus org.kde.kwalletd6 /modules/kwalletd6 isEnabled 2>/dev/null
```

## List wallets

```bash
qdbus org.kde.kwalletd6 /modules/kwalletd6 wallets
```

## Default wallet name

```bash
kreadconfig6 --file kwalletrc --group Wallet --key "Default Wallet" --default kdewallet
```

## Is the default wallet open?

```bash
qdbus org.kde.kwalletd6 /modules/kwalletd6 isOpen "kdewallet"
```

## Notes

- Wallets are stored as `~/.local/share/kwalletd/<name>.kwl` — encrypted with the user's wallet password (or auto-unlocked from PAM via `pam_kwallet`).
- A status check that returns "daemon not running" is normal if no app has touched KWallet this session — calling any of the read/write skills will start it on demand.

> **Stub skill.** Status is implemented; if the user wants richer reporting (last-unlocked, password-strength signals, integration-test against pam_kwallet), flesh out from here.
