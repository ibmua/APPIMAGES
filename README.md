# install-appimage

Utility script to install AppImages by extracting them into a normal directory, optionally creating a desktop entry and a `-latest` symlink.

This approach is useful for Electron-based AppImages that have sandbox permission issues when run directly.

## Quick Reference

### Basic install

```bash
./install-appimage LM-Studio-0.4.2-2-x64.AppImage
```

### Update to a newer version

```bash
./install-appimage LM-Studio-0.5.0-x64.AppImage
```

If the target install directory already exists, the script offers to back it up as `*.backup` before replacing it.

### Common variants

```bash
# custom install directory
./install-appimage MyApp.AppImage ~/Applications/

# skip desktop entry
./install-appimage --no-desktop MyApp.AppImage

# custom app display name in desktop entry
./install-appimage -n "My App Beta" MyApp.AppImage
```

## Usage

```text
install-appimage [OPTIONS] <appimage-file> [install-directory]

Options:
  -d, --desktop          Create desktop entry (default)
  --no-desktop           Skip desktop entry creation
  -n, --name <name>      Custom application name
  -h, --help             Show help message
```

## What It Creates

For `LM-Studio-0.4.2-2-x64.AppImage`, default install location is:

```text
/home/i/Documents/lm-studio-0.4.2-2/
```

Typical outputs:

- Extracted app directory (contains `AppRun` and bundled files)
- Desktop file in `~/.local/share/applications/` (unless `--no-desktop`)
- Symlink like `lm-studio-latest -> lm-studio-0.4.2-2` (when version suffix is detected)

## Behavior Details

### Extraction flow

The script:
1. Makes the AppImage executable.
1. Runs `--appimage-extract` into a temporary directory.
1. Moves `squashfs-root` to the final install path.

### Desktop entry rewrite

When a `.desktop` file exists in the extracted root, the script:

- Rewrites `Exec=` to point at absolute `<install-path>/AppRun`
- Rewrites relative `Icon=` paths to absolute install paths
- Applies `-n/--name` override to `Name=`
- Writes the result to `~/.local/share/applications/`

### `chrome-sandbox` handling

If `chrome-sandbox` exists, the script tries:

```bash
sudo chown root:root <install-path>/chrome-sandbox
sudo chmod 4755 <install-path>/chrome-sandbox
```

If that fails, installation still completes, but the app may need `--no-sandbox` depending on the app.

## Launching

- Desktop launcher from app menu (if desktop entry was created)
- Command line: `<install-path>/AppRun`
- Symlink path: `<install-dir>/<base-name>-latest/AppRun`

## Troubleshooting

### AppImage not found

- Verify the file path.
- Use absolute paths when in doubt.

### Extraction failed

- Ensure the AppImage is executable: `chmod +x <file>.AppImage`
- Confirm the file is valid and not corrupted.

### Desktop entry not visible

- Run `update-desktop-database ~/.local/share/applications/`
- Confirm the desktop file exists in `~/.local/share/applications/`
- Log out/in if your desktop shell caches launchers.

### App does not start

- Run `<install-path>/AppRun` in terminal to inspect errors.
- Ensure `AppRun` is executable.
- If sandbox errors persist, run the app with `--no-sandbox` if supported.

## Uninstall

```bash
# app directory
rm -rf <install-path>

# desktop entry (file name depends on app)
rm ~/.local/share/applications/<app>.desktop

# optional symlink
rm <install-dir>/<base-name>-latest

# refresh desktop db if available
update-desktop-database ~/.local/share/applications/
```

## Example Commands

```bash
./install-appimage LM-Studio-0.4.2-2-x64.AppImage
./install-appimage Bambu_Studio_ubuntu-24.04_PR-9540.AppImage ~/Applications/
./install-appimage --no-desktop "OrcaSlicer_Linux_AppImage_Ubuntu2404_nightly (1).AppImage"
./install-appimage -n "LM Studio Development" LM-Studio-dev.AppImage
```

If everything works, you can safely remove the old `lm-studio-317/` installation and use the new one managed by this script.
