# 🚀 install-appimage

Install AppImages so they show in your app launcher, get proper icons, and can be pinned/favorited.

Works in two modes:

- 📦 `extract` mode (default): unpack AppImage and run `AppRun`
- 🧳 `keep` mode: keep the `.AppImage` file and launch it directly

## 🔎 Common Error This Solves

If you searched for these errors, this tool is for you:

```text
FATAL:sandbox/linux/suid/client/setuid_sandbox_host.cc:166
The SUID sandbox helper binary was found, but is not configured correctly.
You need to make sure that /tmp/.mount_.../chrome-sandbox is owned by root and has mode 4755.
```

Search terms this README targets:

- `LM Studio AppImage not launching Ubuntu`
- `setuid_sandbox_host.cc:166`
- `AppImage chrome-sandbox mode 4755`
- `The SUID sandbox helper binary was found`
- `AppImage desktop icon not showing`
- `How to pin AppImage to dock`

## ✨ What You Get

- ✅ Desktop entry in `~/.local/share/applications`
- ✅ Icon path resolved from inside the AppImage
- ✅ Launch command rewritten to a stable install path
- ✅ Version-agnostic `-latest` symlink for easier updates
- ✅ App launcher visibility and pin-to-dock/favorites support

## ⚡ Quick Start

### 1) Fix LM Studio / Electron sandbox errors fast (recommended)

```bash
./install-appimage LM-Studio-0.4.2-2-x64.AppImage
```

This uses `extract` mode and is best for problematic sandbox cases.

### 2) Keep AppImage file as-is (no full extracted install)

```bash
./install-appimage --keep-appimage MyApp.AppImage ~/Applications/
```

This keeps execution from the `.AppImage` file while still creating desktop launcher + icon.

### 3) Keep mode + custom runtime args (for stubborn apps)

```bash
./install-appimage --keep-appimage -a "--no-sandbox" MyApp.AppImage ~/Applications/
```

## 🧠 How It Works

### 📦 Extract mode (default)

1. Extracts the AppImage.
1. Installs to a stable directory like `/home/i/Documents/myapp-1.2.3/`.
1. Sets desktop `Exec=` to stable `AppRun` path.
1. Resolves icon to an absolute image path.
1. Creates `<app>-latest` symlink when version is detected.

### 🧳 Keep mode

1. Copies `.AppImage` to target install directory.
1. Marks it executable.
1. Builds desktop entry that points `Exec=` to that `.AppImage` (or `<app>-latest.AppImage` symlink).
1. Extracts icon metadata and persists icon under `~/.local/share/icons/install-appimage/`.

## 🛠 Usage

```text
install-appimage [OPTIONS] <appimage-file> [install-directory]

Modes:
  --extract              Extract AppImage into a directory (default)
  --keep-appimage        Keep and run the .AppImage file directly

Options:
  -d, --desktop          Create desktop entry (default)
  --no-desktop           Skip desktop entry creation
  -n, --name <name>      Custom application name
  -a, --args <args>      Extra args appended to desktop Exec command
  -h, --help             Show help message
```

## 📌 Pinning Notes

To pin/favorite in GNOME/KDE/XFCE:

1. Install with desktop entry enabled (default).
1. Launch app from app menu once.
1. Right-click running app icon and choose Pin/Favorite.

If pinning does not appear immediately, refresh desktop database:

```bash
update-desktop-database ~/.local/share/applications/
```

## 🧪 Examples

```bash
# default extract install
./install-appimage LM-Studio-0.4.2-2-x64.AppImage

# keep mode in custom directory
./install-appimage --keep-appimage Bambu_Studio_ubuntu-24.04_PR-9540.AppImage ~/Applications/

# no desktop entry
./install-appimage --no-desktop "OrcaSlicer_Linux_AppImage_Ubuntu2404_nightly (1).AppImage"

# custom launcher name
./install-appimage -n "LM Studio Development" LM-Studio-dev.AppImage
```

## 🆘 Troubleshooting

### `The SUID sandbox helper binary was found, but is not configured correctly`

Use extract mode:

```bash
./install-appimage <your-file>.AppImage
```

It avoids relying on fragile `/tmp/.mount_*` runtime paths and uses a stable install location.

### `/tmp/.mount_.../chrome-sandbox is owned by root and has mode 4755`

Same fix: install via this tool instead of direct `./AppImage` launch.

In extract mode, if `chrome-sandbox` exists, the script tries:

```bash
sudo chown root:root <install-path>/chrome-sandbox
sudo chmod 4755 <install-path>/chrome-sandbox
```

### App icon missing in launcher

- Re-run install so icon resolution is regenerated.
- Confirm desktop file exists in `~/.local/share/applications/`.
- Refresh desktop db:
  `update-desktop-database ~/.local/share/applications/`

### App not visible in menu / not pinnable

- Log out/in once after installation.
- Ensure the `.desktop` file has `Type=Application` and valid `Exec=`.
- Launch from app menu first, then pin from running icon.

## 🧹 Uninstall

```bash
# remove extracted directory OR installed .AppImage
rm -rf <install-path-or-file>

# remove desktop entry
rm ~/.local/share/applications/<app>.desktop

# remove optional latest symlink
rm <install-dir>/<base-name>-latest
rm <install-dir>/<base-name>-latest.AppImage
```
