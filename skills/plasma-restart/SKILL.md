---
name: plasma-restart
description: Restart Plasma components (plasmashell, krunner, kglobalaccel, polkit-kde-authentication-agent, kded) without logging out. Use when a config change isn't taking effect or a Plasma process is misbehaving.
---

# Plasma Component Restart

Restarting individual Plasma components is almost always preferable to logging out.

## plasmashell (taskbar, panels, desktop)

```bash
kquitapp6 plasmashell 2>/dev/null || killall plasmashell
sleep 1
kstart plasmashell
```

If `plasmashell` is wedged and `kquitapp6` hangs, `killall -9 plasmashell` then `kstart plasmashell`.

## krunner

```bash
kquitapp6 krunner 2>/dev/null || killall krunner
# krunner relaunches on demand (Alt+Space)
```

## kglobalaccel (global shortcuts)

```bash
kquitapp6 kglobalaccel 2>/dev/null || killall kglobalaccel
kstart kglobalaccel
```

## kded (background services)

```bash
kquitapp6 kded6 2>/dev/null || killall kded6
kstart kded6
```

## KWin (window manager) — Wayland caveats

On X11, `kwin_x11 --replace &` works. On **Wayland**, KWin is the compositor — restarting it ends the session. Don't restart `kwin_wayland` from inside a Plasma session. Workarounds:

- Trigger `qdbus org.kde.KWin /KWin reconfigure` to re-read config (most changes don't need a full restart).
- For a hard restart, log out and back in.

## Polkit agent

```bash
kquitapp6 polkit-kde-authentication-agent-1 2>/dev/null || killall polkit-kde-authentication-agent-1
/usr/lib/x86_64-linux-gnu/libexec/polkit-kde-authentication-agent-1 &
```

## Notes

- `kquitapp6` is graceful (asks via D-Bus). `killall` is a fallback. `-9` is a last resort.
- After a `plasmashell` restart, ephemeral state (open Klipper history, plasmoid scroll positions) is preserved because they're persisted on quit.
