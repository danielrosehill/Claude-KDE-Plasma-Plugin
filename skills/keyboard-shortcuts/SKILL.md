---
name: keyboard-shortcuts
description: Inspect and audit KGlobalAccel keyboard shortcuts (global hotkeys) on KDE Plasma. Use when the user wants to see what a Meta/Ctrl/Alt combination is bound to, find a free combination, or troubleshoot a shortcut that stopped firing. Reads are reliable; writes on Plasma 6 Wayland are not — see the warning below before offering to bind anything.
---

# KGlobalAccel Shortcuts

Plasma global shortcuts live in `~/.config/kglobalshortcutsrc` and are managed by
the `kglobalaccel` daemon.

> **Reading is reliable. Writing is not.**
> On Plasma 6 Wayland there is currently no verified way to bind a shortcut from
> the command line and have it take effect in the running session. Do not tell the
> user a binding is done after writing the config file — it very likely is not.
> See [Writing](#writing) below.

## Inspect

```bash
# All shortcuts, grouped by component
grep -E '^\[|^[A-Za-z]' ~/.config/kglobalshortcutsrc

# Just one component
kreadconfig6 --file kglobalshortcutsrc --group "krunner.desktop" --key "_launch"

# Which components register shortcuts at all
qdbus6 org.kde.kglobalaccel /kglobalaccel org.kde.KGlobalAccel.allMainComponents
```

### File format

```
ShortcutName=<active>,<default>,<friendly name>
# e.g.
_launch=Alt+Space,Alt+Space,KRunner
```

- `<active>` is what is bound now; `<default>` is what it ships as.
- `<active>=none` means explicitly unbound.
- Multiple combinations are tab-separated within the field: `Meta+R\tAlt+F2`.
- An empty `<active>` with a non-empty `<default>` means "never customised".

### Finding a free combination

To check whether a combination is already taken, grep the whole file for it rather
than reasoning about a single component — bindings are scattered across every
`[component.desktop]` group, and Plasma itself reserves a lot of `Meta+*`.

```bash
grep -n "Meta+Shift+K" ~/.config/kglobalshortcutsrc
```

Remember that this file only holds *global* shortcuts. Per-application accelerators
live in `~/.config/kdeglobals` under `[Shortcuts]` and in per-app `*shortcutsrc`
files, and will not show up here.

## Writing

**Verified not to work on Plasma 6 Wayland** (ThinkPad E15 Gen 3, Plasma 6):

1. **`kwriteconfig6` + D-Bus reload.** The value is written to the file correctly,
   but `kglobalaccel` does not reliably adopt it. In testing the binding only
   became active after a full logout and login. Corroborated by
   [KDE Discuss #17995](https://discuss.kde.org/t/plasma-6-method-to-refresh-kglobalaccel-shortcuts/17995).

   ```bash
   # Writes the file. Does NOT reliably apply.
   kwriteconfig6 --file kglobalshortcutsrc --group "krunner.desktop" \
     --key "_launch" "Alt+Space,Alt+Space,KRunner"
   qdbus6 org.kde.kglobalaccel /kglobalaccel org.kde.KGlobalAccel.reloadConfiguration
   ```

2. **The khotkeys route.** The `khotkeys` KDED module does not exist in Plasma 6.
   `qdbus org.kde.kded6 /modules/khotkeys reread_configuration` has no target;
   `~/.config/khotkeysrc` is a Plasma 5 artefact. Custom command shortcuts moved
   into the Shortcuts KCM.

3. **`.desktop` file with `X-KDE-Shortcuts=`.** Does not register.

### What to do instead

For **custom command shortcuts**, walk the user through the GUI — currently the
only reliable path:

1. System Settings → Shortcuts
2. **+ Add New → Command or Script**
3. Name it, set the command (use an absolute path)
4. **Add…** next to "Custom shortcuts", press the key combination
5. **Apply**

Then, if the result should be reproducible, commit the resulting
`~/.config/kglobalshortcutsrc` to a dotfiles or config repo — capture the outcome
rather than trying to script the input.

For **rebinding an existing application shortcut**, the same KCM has a per-app
search; that also applies immediately.

If the user explicitly accepts a logout, `kwriteconfig6` plus a logout does work,
and is scriptable for machine setup where a logout is happening anyway. Say that
the logout is required rather than presenting it as optional.

Full write-up of what was tried and rejected:
[`custom-keybindings`](https://github.com/danielrosehill/custom-keybindings).

## Troubleshooting a shortcut that stopped firing

1. Confirm it is still in the file — a Plasma upgrade can reset `<active>` to the
   default.
2. Check for a conflict: grep the combination across the whole file; the later
   registrant may have stolen it silently.
3. Confirm the daemon is alive: `pgrep -a kglobalaccel`.
4. On Wayland, Plasma owns the global keymap outright. X11-era tools (`xbindkeys`,
   `xdotool` bindings) do not work and are not a fallback.
5. If the target is a `.desktop` launcher, verify the launcher itself still
   resolves — a broken `Exec=` presents as a dead shortcut.
