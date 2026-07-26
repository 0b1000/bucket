# Field reference

Per-field detail for Scoop app manifests. Read this when a manifest needs fields beyond the common shape shown in SKILL.md's templates (lifecycle scripts, installer objects, persistence, env, psmodule, etc.).

## Contents

- [Required / near-required](#required--near-required)
- [Download](#download)
- [architecture](#architecture)
- [Lifecycle scripts](#lifecycle-scripts)
- [Shims & shortcuts](#shims--shortcuts)
- [Persistence & deps](#persistence--deps)
- [Other](#other)

## Required / near-required

- `version`: the version this manifest installs.
- `description`: one line; omit the app name if it equals the filename.
- `homepage`: program homepage.
- `license`: SPDX identifier (`MIT`, `GPL-3.0`, `LGPL-3.0`, …) or special tokens `Freeware` / `Proprietary` / `Public Domain` / `Shareware` / `Unknown`. Object form `{ "identifier": "...", "url": "..." }` for non-SPDX. Multiple licenses: comma `,`. Dual-licensed: pipe `|`. Hard to enumerate: append `,...`. List non-open-source licenses first.

## Download

- `url`: string or array. Append a URL fragment starting with `#/` to rename the saved file, e.g. `program.exe#/dl.7z` — Scoop saves it as `dl.7z` and extracts it, bypassing NSIS installers / UAC / registry writes. The same fragment also stabilizes a per-arch binary name (e.g. `zread-windows-x64.exe#/zread.exe`) so `bin`/`shortcuts` targets don't vary by arch.
- `hash`: string or array aligned with `url`. SHA256 by default; prefix `sha1:` / `md5:` / `sha512:` for other algorithms.
- `extract_dir`: pull a specific subdirectory out of the archive.
- `extract_to`: extract ALL content into the given directory (distinct from `extract_dir`).
- `innosetup`: boolean `true` (unquoted) for InnoSetup installers.

## architecture

Wraps `32bit` / `64bit` / `arm64`. Each may hold `url`, `hash`, `extract_dir`, `extract_to`, `pre_install`, `installer`, `post_install`, `pre_uninstall`, `uninstaller`, `post_uninstall`, `env_add_path`, `env_set`, `bin`, `shortcuts`, `persist`. Use when downloads or launchers differ by arch (canonical example: Main's `7zip.json`). For a shared download with arch-specific `bin`/`shortcuts` only, keep `url` + `hash` top-level and put the launchers per-arch (see Extras' `sysinternals.json`).

## Lifecycle scripts

- `pre_install` / `post_install` / `pre_uninstall` / `post_uninstall`: string or array of strings. Variables: `$dir`, `$persist_dir`, `$version`.
- `installer` / `uninstaller`: object `{ file, script, args, keep }`.
  - `file`: installer exe (defaults to the last downloaded URL).
  - `script`: string or array run instead of `file`.
  - `args`: array passed to the installer.
  - `keep`: `"true"` to retain the installer for later uninstall.
  - Script/args vars: `$fname` (last downloaded file), `$manifest`, `$architecture`, `$dir`.
  - `installer` runs on both `scoop install` and `scoop update`; `uninstaller` on both `scoop uninstall` and `scoop update`.
  - To unwrap a wrapped NSIS/7z payload, call `Expand-7ZipArchive` in `pre_install` (see Extras' `heynote.json`, `another-redis-desktop-manager.json`) or `installer.script` (see Extras' `calibre.json`).

## Shims & shortcuts

- `bin`: string or array of executables placed on PATH. Alias shim form: `["program.exe", "alias", "--args"]`. A single alias must be wrapped in an outer array: `"bin": [["program.exe", "alias"]]`.
- `shortcuts`: array of pairs `[target, label]`; optional 3rd = start parameters, 4th = icon path. Label supports subdirectories: `"SubDir\\App"`. These create Start-menu shortcuts.
- `env_add_path`: directory relative to the install dir; must be inside it.
- `env_set`: map of environment variables.

## Persistence & deps

- `persist`: string or array of files/directories persisted in the app's data directory.
- `depends`: runtime dependencies auto-installed.
- `suggest`: `{ "Feature": ["app1", "app2"] }` — optional complementary apps.

## Other

- `notes`: string or array shown after install.
- `##`: comment string or array (replaces the deprecated `_comment`).
- `psmodule`: `{ "name": "..." }` to install as a PowerShell module.
- Deprecated: `_comment` (use `##`), `msi` (treat `.msi` like `.zip`).
