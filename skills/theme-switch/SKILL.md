---
name: theme-switch
description: Apply Plasma look-and-feel packages, color schemes, icon themes, cursor themes, and Plasma styles via the plasma-apply-* CLI tools. Use when the user wants to script theme changes or switch between light/dark/custom presets.
---

# Theme Switching

Plasma 6 ships dedicated CLI applicators — prefer these over editing `kdeglobals` directly.

## Look-and-feel package (the "Global Theme")

Bundles colour scheme + Plasma style + icon theme + window deco + cursor + wallpaper.

```bash
# List installed
plasma-apply-lookandfeel --list

# Apply
plasma-apply-lookandfeel --apply org.kde.breezedark.desktop
```

## Colour scheme only

```bash
plasma-apply-colorscheme BreezeDark
plasma-apply-colorscheme --list-schemes
```

## Plasma style (widget chrome)

```bash
plasma-apply-desktoptheme breeze-dark
plasma-apply-desktoptheme --list-themes
```

## Cursor theme

```bash
plasma-apply-cursortheme breeze_cursors
plasma-apply-cursortheme --list-themes
```

## Wallpaper

Per-monitor via `plasmashell` scripting (see `plasmoid-manage` for the pattern). For a quick global swap:

```bash
plasma-apply-wallpaperimage /path/to/wallpaper.jpg
```

## Icon theme

No dedicated applicator — write `kdeglobals` `[Icons]` `Theme=` via `kwriteconfig-edit`, then `kbuildsycoca6` to refresh:

```bash
kwriteconfig6 --file kdeglobals --group Icons --key Theme "breeze-dark"
kbuildsycoca6
```

## Notes

- Plasma 5 equivalents: `lookandfeeltool -a <pkg>`, no `plasma-apply-*` shortcuts.
- GTK app theming is handled separately by `kde-gtk-config` — check `~/.config/gtk-3.0/settings.ini`.
- Theme presets and saved combos can be stored in `${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/kde-plasma/data/theme-presets/`.
