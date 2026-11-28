---
name: add-scoop-app
description: >-
  Standardized workflow for adding a new application manifest to this Scoop bucket.
  Use when the user provides a software name, version, or download URL (such as a
  GitHub release asset or installer/archive) and asks to add it to the bucket.
---

# Add Scoop App

Follow this standardized workflow using Scoop's built-in scripts and CLI tools to add, verify, and commit a new package manifest in `bucket/`.

## Prerequisites & Conventions

- **Package name**: lowercase with hyphens (e.g., `altium-designer-mcp`).
- **Manifest path**: `bucket/<pkg>.json`.
- **Scoop scripts**: Always prefer the repository's `.\bin\*.ps1` wrappers (`checkhashes.ps1`, `formatjson.ps1`, `checkver.ps1`, `test.ps1`) and `scoop` CLI over manual hashing or custom scripts.

## Workflow Steps

### 1. Prepare Manifest from Template

Copy an existing manifest in `bucket/` that matches the app's distribution type (e.g., GitHub release CLI tool, portable GUI app, or archive with `architecture.64bit`), or create `bucket/<pkg>.json` with:

- `version`: Target version string (without leading `v`).
- `description`: Concise summary of the application.
- `homepage`: Project homepage or GitHub repository URL.
- `license`: SPDX license identifier (e.g., `MIT`, `Apache-2.0`, `GPL-3.0`, `Proprietary`, `Freeware`).
- `architecture.64bit.url` (or top-level `url`): Direct download URL.
- `architecture.64bit.hash` (or top-level `hash`): Placeholder 64-char hex string `"0000000000000000000000000000000000000000000000000000000000000000"` (do **not** leave `""`, otherwise `checkhashes.ps1` reports a URLs/hashes count mismatch).
- `checkver`: `"github"` for GitHub releases, or appropriate `url` + `jsonpath`/`regex`.
- `autoupdate`: Matching URL template using `$version`.

### 2. Download & Compute Hash

Run Scoop's `checkhashes.ps1` wrapper to download the artifact into Scoop's cache and update the manifest hash automatically:

```powershell
.\bin\checkhashes.ps1 -App <pkg> -Update
```

### 3. Inspect Archive & Configure Entrypoints

If the downloaded file is an archive (`.zip`, `.7z`, `.tar.gz`, etc.), resolve Scoop's cache directory dynamically (respecting `SCOOP_CACHE`, `scoop config cache_path`, and the Scoop root) and list its contents with `7z`:

```powershell
$scoopRoot = Split-Path (Split-Path (Split-Path (scoop prefix scoop)))
$cacheDir = @($env:SCOOP_CACHE, (scoop config cache_path 6>$null), (Join-Path $scoopRoot 'cache')) |
    Where-Object { $_ -and (Test-Path $_) } |
    Select-Object -First 1
Get-ChildItem (Join-Path $cacheDir '*<pkg>*') | ForEach-Object { 7z l $_.FullName }
```

Based on the archive contents and application behavior, configure entrypoints, extraction, and persistence in `bucket/<pkg>.json`:

- **Entrypoints (`bin` / `shortcuts`)**:
  - **CLI / MCP / terminal tools**: `"bin": "<exe-or-relative-path>"` (or array for multiple binaries).
  - **GUI applications**: `"shortcuts": [["<exe-or-relative-path>", "<Display Name>"]]`.
  - **Both CLI & GUI** (e.g., `bucket/mpv.net-dw.json`): include both `"bin"` and `"shortcuts"` when the app supports command-line invocation as well as GUI launch.
- **Unwrapping subdirectories (`extract_dir`)** (see `bucket/mpv.net-dw.json`):
  - If the archive (or `#/dl.7z` installer) wraps all files inside a single top-level folder, set `"extract_dir": "<folder>"`.
  - If `<folder>` includes the version number (e.g., `"mpv.net-DW v2.1.0"`), also add `"extract_dir": "...$version..."` under `"autoupdate"`.
- **Persistence (`persist` & `pre_install`)** (see `bucket/mpv.net-dw.json`, `bucket/hoyo-gacha.json`):
  - Add `"persist"` for config/data folders or files stored next to the executable (e.g., `"portable_config"`, `"HoYo.Gacha.v1.db"`).
  - If a persisted file or directory is created at runtime and does **not** exist inside the archive/download, add a `"pre_install"` script so Scoop can link it on first install:
    - For a file (like `HoYo.Gacha.v1.db` in `bucket/hoyo-gacha.json`):
      `if (!(Test-Path "$persist_dir\<file>")) { New-Item -Path "$dir\<file>" -ItemType File | Out-Null }`
    - For a directory (like `step` in `bucket/lceda-step-downloader.json`):
      `if (!(Test-Path "$persist_dir\<dir>")) { New-Item -Path "$dir\<dir>" -ItemType Directory | Out-Null }`

### 4. Format & Validate Manifest

Run Scoop's formatter and test/checkver scripts as applicable:

```powershell
.\bin\formatjson.ps1 -App <pkg>
.\bin\checkver.ps1 -App <pkg>
```

### 5. Local Install & Verify

Install directly from the local manifest and verify the installed binary/shortcut works:

```powershell
scoop install .\bucket\<pkg>.json
# For CLI tools, verify the shim runs (e.g. <pkg> --version or <pkg> --help)
```

Confirm `scoop install` reports `Checking hash... OK.` and `was installed successfully!`.

### 6. Confirm, Uninstall & Commit

**Stop and ask the user for confirmation** before uninstalling and committing, so the user can manually inspect or test the installed app if needed.

Once the user confirms, uninstall cleanly and commit the new manifest following the repository's commit message style (`<pkg>: Add version <version>`):

```powershell
scoop uninstall <pkg>
git add bucket/<pkg>.json
git commit -m "<pkg>: Add version <version>"
```
