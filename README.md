# 🚀 install-appimage

Install AppImages so they show up in your app launcher, get proper icons, and can be pinned or favorited — on any Linux desktop.

Two modes depending on what you need:

- 📦 **Extract** (default) — unpacks the AppImage and runs `AppRun` from a stable directory. Fixes Electron sandbox errors.
- 🧳 **Keep** — leaves the `.AppImage` file intact and launches it directly. Still gets a desktop entry and icon.

## 🔎 Common Errors This Solves

If you landed here searching for one of these, you're in the right place:

```text
FATAL:sandbox/linux/suid/client/setuid_sandbox_host.cc:166
The SUID sandbox helper binary was found, but is not configured correctly.
You need to make sure that /tmp/.mount_.../chrome-sandbox is owned by root and has mode 4755.
```

Related searches:

- `AppImage not launching Ubuntu`
- `setuid_sandbox_host.cc:166`
- `AppImage chrome-sandbox mode 4755`
- `The SUID sandbox helper binary was found`
- `AppImage desktop icon not showing`
- `How to pin AppImage to dock`
- `Electron AppImage sandbox error Linux`

Just run install-appimage in extract mode (the default). It places the app in a stable directory and fixes `chrome-sandbox` permissions automatically — no more fragile `/tmp/.mount_*` paths.

## ⚡ Quick Start

### 1) Extract install (fixes sandbox errors)

```bash
./install-appimage MyApp-1.2.3-x64.AppImage
```

Best for Electron apps that hit the `chrome-sandbox` / `setuid_sandbox_host` error.

### 2) Keep the .AppImage file as-is

```bash
./install-appimage --keep-appimage MyApp.AppImage ~/Applications/
```

Keeps execution from the `.AppImage` file while still creating a desktop launcher and icon.

### 3) Keep mode + custom runtime args (for stubborn apps)

```bash
./install-appimage --keep-appimage -a "--no-sandbox" MyApp.AppImage ~/Applications/
```

## ✨ What You Get

- ✅ Desktop entry in `~/.local/share/applications/`
- ✅ Icon resolved from inside the AppImage
- ✅ Stable install path (no more `/tmp/.mount_*` breakage)
- ✅ Version-agnostic `-latest` symlink for painless updates
- ✅ App shows up in your launcher and can be pinned/favorited

## 🧠 How It Works

### 📦 Extract mode (default)

1. Extracts the AppImage contents.
2. Installs to a stable directory like `~/Documents/myapp-1.2.3/`.
3. Points the desktop entry's `Exec=` at the stable `AppRun` path.
4. Resolves the icon to an absolute path so the launcher finds it.
5. Creates a `<app>-latest` symlink when a version number is detected.

### 🧳 Keep mode

1. Copies the `.AppImage` to the target directory and marks it executable.
2. Builds a desktop entry pointing `Exec=` at the `.AppImage` (or a `-latest.AppImage` symlink).
3. Extracts and persists the icon under `~/.local/share/icons/install-appimage/`.

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

## 🧪 Examples

```bash
# default extract install
./install-appimage Bambu_Studio_ubuntu-24.04.AppImage

# keep mode, custom directory
./install-appimage --keep-appimage OrcaSlicer-1.9.0-x64.AppImage ~/Applications/

# skip the desktop entry
./install-appimage --no-desktop SomeToolkit.AppImage

# custom launcher name
./install-appimage -n "My Custom Name" SomeApp-nightly.AppImage
```

## 📌 Pinning to Dock

After installing:

1. Open the app from your launcher/app menu.
2. Right-click the running app's icon and choose **Pin** or **Add to Favorites**.

If it doesn't appear right away, refresh the desktop database:

```bash
update-desktop-database ~/.local/share/applications/
```

## 🆘 Troubleshooting

### `The SUID sandbox helper binary was found, but is not configured correctly`

Use extract mode (the default). It installs to a stable path and fixes `chrome-sandbox` ownership and permissions automatically:

```bash
./install-appimage YourApp.AppImage
```

### `/tmp/.mount_.../chrome-sandbox is owned by root and has mode 4755`

Same fix — install via this tool instead of launching the `.AppImage` directly.

In extract mode, if `chrome-sandbox` exists, the script runs:

```bash
sudo chown root:root <install-path>/chrome-sandbox
sudo chmod 4755 <install-path>/chrome-sandbox
```

If the automatic fix fails, the script will tell you. You can also pass `--no-sandbox` as a workaround:

```bash
./install-appimage --keep-appimage -a "--no-sandbox" YourApp.AppImage
```

### App icon missing in launcher

- Re-run install-appimage so icon resolution runs again.
- Check that the desktop file exists: `ls ~/.local/share/applications/`
- Refresh: `update-desktop-database ~/.local/share/applications/`

### App not visible in menu / not pinnable

- Log out and back in after installing.
- Make sure the `.desktop` file has `Type=Application` and a valid `Exec=` line.
- Launch from the app menu first, then pin from the running icon.

## 🧹 Uninstall

```bash
# remove the installed app (directory or .AppImage file)
rm -rf <install-path>

# remove the desktop entry
rm ~/.local/share/applications/<app>.desktop

# remove the -latest symlink if one was created
rm <install-dir>/<app>-latest
rm <install-dir>/<app>-latest.AppImage
```
