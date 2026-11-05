---
name: install-kde-dev-tools
description: Use when the user wants to install the KDE Plasma 6 / Qt 6 development toolchain on a Linux workstation — Qt Creator, KDevelop, plasma-sdk (Cuttlefish, plasmathemeexplorer, plasmoidviewer), Kirigami Gallery, QML tooling, and the build/packaging utilities needed for plasmoid development. Triggers on phrases like "install KDE dev tools", "set up Plasma 6 dev environment", "install Qt Creator", "plasma-sdk", "Cuttlefish", "Kirigami gallery". Assumes a KDE Plasma system; detects distro and uses the native package manager.
---

# Install KDE Plasma 6 / Qt 6 Development Tools

Provision a workstation for KDE Plasma 6 plasmoid and Qt 6 application development. Detects the distro and installs the canonical packages.

## Detect distro and package manager first

```bash
. /etc/os-release && echo "$ID $VERSION_ID"
which apt dnf zypper pacman 2>/dev/null
```

Branch on `$ID`:

- `ubuntu` / `debian` / `kubuntu` / `neon` → `apt`
- `fedora` / `nobara` → `dnf`
- `opensuse-tumbleweed` / `opensuse-leap` → `zypper`
- `arch` / `manjaro` / `endeavouros` → `pacman` (+ AUR helper if present)

Confirm Plasma 6: `plasmashell --version` should report 6.x. If 5.x, tell the user this toolchain targets Plasma 6 and ask whether they want to proceed anyway.

## Tool inventory

What to install, grouped by purpose. Skip anything the user explicitly excludes.

### Core build toolchain (always)

- **Qt 6 base + qmake6 / cmake** — required to build anything
- **Extra CMake Modules (ECM)** — KDE Frameworks 6 build glue
- **KDE Frameworks 6 dev headers** — KConfig, KCoreAddons, KI18n, KIO, KWindowSystem, etc.
- **Plasma framework dev headers** — for plasmoid C++ data engines (rare, but needed for non-pure-QML plasmoids)
- **`kpackagetool6`** — install/uninstall plasmoids and other KPackages
- **gcc / g++ / make / git**

### IDEs & editors

- **Qt Creator** — Qt's official IDE, best QML editor and live preview
- **KDevelop** — KDE's IDE, great for KF6/Plasma C++ projects
- **Kate** — lightweight QML editing with LSP support (usually preinstalled on Plasma)

### plasma-sdk (essential for plasmoid work)

The `plasma-sdk` package bundles:

- **`plasmoidviewer6`** — live-preview a plasmoid in isolation
- **Cuttlefish** — icon browser; pick freedesktop icon names for `metadata.json`
- **`plasmathemeexplorer`** — inspect SVG theme elements and Plasma theme structure
- **`plasmaengineexplorer`** — browse Plasma data engines (legacy, still useful)
- **`lookandfeelexplorer`** — preview/edit Look-and-Feel packages

### QML / Qt tooling

- **`qmllint` / `qmlformat`** — lint and format QML (ships with Qt 6 declarative tools)
- **`qml6` runtime** — run standalone .qml files
- **Kirigami runtime + Gallery** — `kirigami-gallery` is a live demo of every Kirigami component, indispensable as a visual reference

### Optional but useful

- **`gammaray`** — Qt application introspector (inspect QML object trees of a running app)
- **`qttools6-dev-tools`** / **Qt Linguist** — translation workflow
- **`appstream`** + **`appstreamcli`** — validate `.metainfo.xml` for KDE Store submissions

## Install commands per distro

### Ubuntu / Debian / Kubuntu / KDE Neon

```bash
sudo apt update
sudo apt install -y \
  build-essential cmake extra-cmake-modules git \
  qt6-base-dev qt6-declarative-dev qt6-tools-dev qt6-tools-dev-tools \
  qml6-module-qtquick qml6-module-qtquick-controls \
  libkf6config-dev libkf6coreaddons-dev libkf6i18n-dev \
  libkf6kio-dev libkf6windowsystem-dev libkf6plasma-dev \
  kpackagetool6 \
  plasma-sdk \
  kirigami-gallery \
  qtcreator kdevelop \
  gammaray
```

Note: on older Ubuntu releases the package names may still be `libkf5*` / `qt5-*`. If `apt-cache search libkf6plasma-dev` returns nothing, the user is on a release that doesn't ship KF6 yet — recommend KDE Neon or Kubuntu 24.04+ and stop.

### Fedora

```bash
sudo dnf install -y \
  @development-tools cmake extra-cmake-modules git \
  qt6-qtbase-devel qt6-qtdeclarative-devel qt6-qttools-devel \
  kf6-kconfig-devel kf6-kcoreaddons-devel kf6-ki18n-devel \
  kf6-kio-devel kf6-kwindowsystem-devel kf6-plasma-devel \
  plasma-sdk kirigami-gallery \
  qt-creator kdevelop \
  gammaray
```

### openSUSE Tumbleweed

```bash
sudo zypper install -y -t pattern devel_C_C++ devel_qt6
sudo zypper install -y \
  cmake extra-cmake-modules git \
  plasma6-sdk kirigami-gallery \
  qt-creator kdevelop6 \
  gammaray
```

KF6/Plasma 6 dev packages on Tumbleweed are typically pulled in transitively by `plasma6-sdk`; confirm with `zypper search -i kf6-`.

### Arch / Manjaro / EndeavourOS

```bash
sudo pacman -S --needed \
  base-devel cmake extra-cmake-modules git \
  qt6-base qt6-declarative qt6-tools \
  kconfig kcoreaddons ki18n kio kwindowsystem plasma-framework \
  plasma-sdk kirigami-gallery \
  qtcreator kdevelop \
  gammaray
```

`plasma-sdk` on Arch already includes `plasmoidviewer`, Cuttlefish, plasmathemeexplorer, and lookandfeelexplorer.

## Verification

After install, verify the toolchain:

```bash
qmake6 --version
cmake --version
kpackagetool6 --version
plasmoidviewer6 --help | head -3
which cuttlefish plasmathemeexplorer kirigami-gallery
qtcreator --version 2>/dev/null || echo "Qt Creator: launch via menu"
kdevelop --version 2>/dev/null
```

If any of the executables are missing, the corresponding package didn't install — tell the user which one and offer to investigate.

## Output convention

When asked to set up the toolchain:

1. Detect distro and Plasma version. If not Plasma 6 on a supported distro, stop and ask.
2. Show the user the exact `apt`/`dnf`/`zypper`/`pacman` command you'll run, grouped so they can comment out anything unwanted.
3. Run the install (it will prompt for sudo).
4. Run the verification block and report any missing tools.
5. Recommend next steps: launch `kirigami-gallery` to browse components, `cuttlefish` to pick icons, and the `kde-plasmoid-dev` skill in this same plugin to scaffold a first widget.
