---
name: create-scoop-manifest
description: Create a new Scoop app manifest in this bucket repo. Use when the user asks to add, scaffold, or draft a new app/package manifest JSON, or to add a new app/tool to the bucket. Encodes this repo's canonical field order, the official Scoop manifest schema, checkver/autoupdate patterns, substitution variables, and the bin/ validation scripts.
---

# Create a Scoop manifest

## When to use

Triggered when adding a new app to this bucket. Output: `bucket/<name>.json`.

- NOT for the `deprecated/` folder — move an existing manifest there only when retiring it.
- NOT for editing existing manifests — use the normal edit flow unless explicitly asked.
- Always reorder any manifest you touch to the canonical field order below. The repo's `bin/formatjson.ps1` only normalizes whitespace/values; it does **not** reorder keys.

## File placement & naming

- Path: `bucket/<name>.json`.
- Name: lowercase, hyphen-separated, matching the app's canonical id (e.g. `nat-type-tester`, `mpv.net-dw`). Dots are allowed when matching a real product name.
- Retiring an app: move `bucket/<name>.json` → `deprecated/<name>.json`.

## Canonical field order

Aligned with ScoopInstaller/Main conventions.

Top-level:

```
version, description, homepage, license, notes,
url, hash, extract_dir, extract_to,
architecture,
pre_install, installer, post_install, pre_uninstall, uninstaller, post_uninstall,
env_add_path, env_set, bin, shortcuts, persist,
checkver, autoupdate
```

`architecture.<32bit|64bit|arm64>`:

```
url, hash, extract_dir, extract_to,
pre_install, installer, post_install, uninstaller,
bin, shortcuts, persist
```

`autoupdate`:

```
url, hash, extract_dir, extract_to, architecture
```

`autoupdate.architecture.<arch>` mirrors the per-arch order above.

## Templates

### Template A — single-arch, GitHub releases (common GUI-app pattern; see Extras' heynote.json)

```json
{
    "version": "1.0.0",
    "description": "One-line description without the app name.",
    "homepage": "https://github.com/<owner>/<repo>",
    "license": "MIT",
    "architecture": {
        "64bit": {
            "url": "https://github.com/<owner>/<repo>/releases/download/v1.0.0/<app>-x64.zip",
            "hash": "<sha256>"
        }
    },
    "shortcuts": [
        [
            "<app>.exe",
            "<App Name>"
        ]
    ],
    "checkver": "github",
    "autoupdate": {
        "architecture": {
            "64bit": {
                "url": "https://github.com/<owner>/<repo>/releases/download/v$version/<app>-x64.zip"
            }
        }
    }
}
```

### Template B — dual-arch with per-arch extract_dir (mirrors Main's nodejs.json)

```json
{
    "version": "0.3.5",
    "description": "...",
    "homepage": "https://github.com/<owner>/<repo>",
    "license": "LGPL-3.0",
    "architecture": {
        "64bit": {
            "url": "https://github.com/<owner>/<repo>/releases/download/v0.3.5/<app>-win64.7z",
            "hash": "<sha256>",
            "extract_dir": "<app>-win64"
        },
        "32bit": {
            "url": "https://github.com/<owner>/<repo>/releases/download/v0.3.5/<app>-win32.7z",
            "hash": "<sha256>",
            "extract_dir": "<app>-win32"
        }
    },
    "bin": "<app>.exe",
    "shortcuts": [
        [
            "<app>.exe",
            "<App Name>"
        ]
    ],
    "checkver": "github",
    "autoupdate": {
        "architecture": {
            "64bit": {
                "url": "https://github.com/<owner>/<repo>/releases/download/v$version/<app>-win64.7z",
                "extract_dir": "<app>-win64"
            },
            "32bit": {
                "url": "https://github.com/<owner>/<repo>/releases/download/v$version/<app>-win32.7z",
                "extract_dir": "<app>-win32"
            }
        }
    }
}
```

### Minimal flat (no architecture block)

```json
{
    "version": "1.0.0",
    "description": "...",
    "homepage": "https://example.com/",
    "license": "Freeware",
    "url": "https://example.com/app.zip",
    "hash": "<sha256>",
    "shortcuts": [
        [
            "app.exe",
            "App"
        ]
    ]
}
```

### Template C — dual-arch (64bit + arm64), raw `.exe`, product homepage (mirrors `bucket/zread.json`)

For single-binary GitHub releases with **no published checksum**: omit `hash` entirely (manifest + `autoupdate`); Excavator fills hashes in CI. The `#/` fragment renames each per-arch binary to a stable `bin` target.

```json
{
    "version": "0.2.13",
    "description": "Turns your local codebase into readable docs.",
    "homepage": "https://z.ai/",
    "license": "Proprietary",
    "architecture": {
        "64bit": {
            "url": "https://github.com/<owner>/<repo>/releases/download/v0.2.13/<app>-windows-x64.exe#/<app>.exe"
        },
        "arm64": {
            "url": "https://github.com/<owner>/<repo>/releases/download/v0.2.13/<app>-windows-arm64.exe#/<app>.exe"
        }
    },
    "bin": "<app>.exe",
    "checkver": {
        "github": "https://github.com/<owner>/<repo>"
    },
    "autoupdate": {
        "architecture": {
            "64bit": {
                "url": "https://github.com/<owner>/<repo>/releases/download/v$version/<app>-windows-x64.exe#/<app>.exe"
            },
            "arm64": {
                "url": "https://github.com/<owner>/<repo>/releases/download/v$version/<app>-windows-arm64.exe#/<app>.exe"
            }
        }
    }
}
```

## Field reference

### Required / near-required

- `version`: the version this manifest installs.
- `description`: one line; omit the app name if it equals the filename.
- `homepage`: program homepage.
- `license`: SPDX identifier (`MIT`, `GPL-3.0`, `LGPL-3.0`, …) or special tokens `Freeware` / `Proprietary` / `Public Domain` / `Shareware` / `Unknown`. Object form `{ "identifier": "...", "url": "..." }` for non-SPDX. Multiple licenses: comma `,`. Dual-licensed: pipe `|`. Hard to enumerate: append `,...`. List non-open-source licenses first.

### Download

- `url`: string or array. Append a URL fragment starting with `#/` to rename the saved file, e.g. `program.exe#/dl.7z` — Scoop saves it as `dl.7z` and extracts it, bypassing NSIS installers / UAC / registry writes. The same fragment also stabilizes a per-arch binary name (e.g. `zread-windows-x64.exe#/zread.exe`) so `bin`/`shortcuts` targets don't vary by arch.
- `hash`: string or array aligned with `url`. SHA256 by default; prefix `sha1:` / `md5:` / `sha512:` for other algorithms.
- `extract_dir`: pull a specific subdirectory out of the archive.
- `extract_to`: extract ALL content into the given directory (distinct from `extract_dir`).
- `innosetup`: boolean `true` (unquoted) for InnoSetup installers.

### architecture

Wraps `32bit` / `64bit` / `arm64`. Each may hold `url`, `hash`, `extract_dir`, `extract_to`, `pre_install`, `installer`, `post_install`, `pre_uninstall`, `uninstaller`, `post_uninstall`, `env_add_path`, `env_set`, `bin`, `shortcuts`, `persist`. Use when downloads or launchers differ by arch (canonical example: Main's `7zip.json`). For a shared download with arch-specific `bin`/`shortcuts` only, keep `url` + `hash` top-level and put the launchers per-arch (see Extras' `sysinternals.json`).

### Lifecycle scripts

- `pre_install` / `post_install` / `pre_uninstall` / `post_uninstall`: string or array of strings. Variables: `$dir`, `$persist_dir`, `$version`.
- `installer` / `uninstaller`: object `{ file, script, args, keep }`.
  - `file`: installer exe (defaults to the last downloaded URL).
  - `script`: string or array run instead of `file`.
  - `args`: array passed to the installer.
  - `keep`: `"true"` to retain the installer for later uninstall.
  - Script/args vars: `$fname` (last downloaded file), `$manifest`, `$architecture`, `$dir`.
  - `installer` runs on both `scoop install` and `scoop update`; `uninstaller` on both `scoop uninstall` and `scoop update`.
  - To unwrap a wrapped NSIS/7z payload, call `Expand-7ZipArchive` in `pre_install` (see Extras' `heynote.json`, `another-redis-desktop-manager.json`) or `installer.script` (see Extras' `calibre.json`).

### Shims & shortcuts

- `bin`: string or array of executables placed on PATH. Alias shim form: `["program.exe", "alias", "--args"]`. A single alias must be wrapped in an outer array: `"bin": [["program.exe", "alias"]]`.
- `shortcuts`: array of pairs `[target, label]`; optional 3rd = start parameters, 4th = icon path. Label supports subdirectories: `"SubDir\\App"`. These create Start-menu shortcuts.
- `env_add_path`: directory relative to the install dir; must be inside it.
- `env_set`: map of environment variables.

### Persistence & deps

- `persist`: string or array of files/directories persisted in the app's data directory.
- `depends`: runtime dependencies auto-installed.
- `suggest`: `{ "Feature": ["app1", "app2"] }` — optional complementary apps.

### Other

- `notes`: string or array shown after install.
- `` `##` ``: comment string or array (replaces the deprecated `_comment`).
- `psmodule`: `{ "name": "..." }` to install as a PowerShell module.
- Deprecated: `_comment` (use `` `##` ``), `msi` (treat `.msi` like `.zip`).

## checkver patterns

Every manifest needs `checkver` + `autoupdate`; a version-only manifest is unfinished. Pick a `checkver` form:

- `"github"` — homepage = repo URL; matches `/releases/tag/(?:v|V)?([\d.]+)` and **ignores pre-releases**.
- `{ "github": "https://github.com/<owner>/<repo>" }` — homepage differs from the repo.
- Bare regex string — matched against the `homepage` source.
- `{ "url": "<page>", "regex": "..." }` — scrape a different page.
- `{ "url": "<endpoint>", "jsonpath": "$.latest" }` (alias `jp`) — JSON endpoint + JSONPath.
- `{ "url": "<endpoint>", "jsonpath": "$.stable", "regex": "v([\\d.]+)" }` — JSONPath extracts a string, then regex derives the version.
- `{ "url": "<page>", "xpath": "..." }` — XPath.
- Extras: `reverse: true` (match the last occurrence), `replace` (rewrite the matched value), `useragent`, `script` (PowerShell for multi-hop/complex cases).

**Pitfall — `homepage` ≠ repo:** the string form `"github"` derives the repo from `homepage`. If `homepage` is a product/marketing page rather than the GitHub repo (e.g. `https://z.ai/`), it fails with `checkver expects the homepage to be a github repository`. Use the object form `{ "github": "<repo-url>" }` and keep `homepage` as the product page. See `bucket/zread.json`.

`checkver` and `autoupdate` always appear together; if you can't yet derive the version, leave a TODO.

## autoupdate

Mirrors the manifest: most fields can live in `autoupdate` — `url`, `hash`, `extract_dir`, `extract_to`, `bin`, `shortcuts`, `persist`, `installer`, `post_install`, `env_add_path`, `env_set`, `license`, `note`, `psmodule`. Set globally (applies to every arch) or per-arch under `architecture.64bit` / `32bit`. A global field updates each arch's corresponding field.

### hash sub-property

`autoupdate.hash`:

- `mode`: `extract` (default; plain text/webpage via regex) | `json` | `xpath` | `rdf` | `metalink` | `fosshub` (auto) | `sourceforge` (auto) | `download` (fallback: download and hash locally).
- `url`: template for the hash source. Under `mode: extract`, built-in regexes apply: `^([a-fA-F0-9]+)$` and `([a-fA-F0-9]{32,128})[\x20\t]+.*$basename(?:[\x20\t]+\d+)?`.
- `regex` / `find`: custom regex (supports hash variables like `$sha256`).
- `jsonpath` / `jp`, `xpath`: for JSON / XML sources.

FossHub / SourceForge download URLs are auto-detected, so no `hash` block is needed.

**Upstream publishes a checksum?** Wire `autoupdate.hash` to PULL it (a SHA256SUMS file, checksums page, or API field) — Scoop fetches the hash without downloading the binary. Fast.

**Upstream publishes no checksum?** (common for single-binary GitHub releases like `zread`): **omit `hash` entirely** — from both the manifest and `autoupdate`. Do NOT compute hashes by hand (`Get-FileHash`) and do NOT run `checkver -ForceUpdate` to bootstrap; both download every asset and waste time. Ship the manifest hashless. Scoop install will warn "No hash in manifest" but still install; Excavator will compute hashes in CI on future releases via autoupdate's download fallback (when `url` autoupdates, `Invoke-AutoUpdate` always re-adds `hash`).

## Substitution variables

- Version: `$version`, `$underscoreVersion`, `$dashVersion`, `$cleanVersion`, `$majorVersion` / `$minorVersion` / `$patchVersion` / `$buildVersion`, `$matchHead` (first 2–3 dotted segments), `$matchTail`, `$preReleaseVersion`.
- URL: `$url` (no fragment), `$baseurl` (no filename or fragment), `$basename` (filename, ignores `#/...`).
- Hash: `$md5`, `$sha1`, `$sha256`, `$sha512`, `$checksum` (any), `$base64`.
- Captured (from `checkver.regex` groups): use `$match1` / `$matchName` inside `autoupdate`; use `${1}` / `${name}` inside `checkver.replace`. Named-group variable names are camelCase with only the first letter uppercase (e.g. `$matchVersion`, `$matchShort`).

## Validation (run from repo root, in this order)

All scripts delegate to the local Scoop install (`scoop prefix scoop`); Scoop must be installed.

1. `.\bin\formatjson.ps1`            — normalize formatting/values, confirm JSON parses.
2. `.\bin\checkver.ps1 <app>`        — confirm the detected version matches `version`.
3. `.\bin\checkurls.ps1 <app>`       — every download URL resolves.
4. `.\bin\checkhashes.ps1 <app>`     — only when the manifest ships `hash` values; for hashless manifests it errors with "URLS and hashes count mismatch" (expected — skip).

**Do NOT bootstrap hashes with `checkver -ForceUpdate` or hand-rolled `Get-FileHash`.** If upstream provides no checksum, ship the manifest hashless (see autoupdate § above); Excavator fills hashes in CI. `checkhashes.ps1 -Update` fixes *wrong* hashes in place but cannot bootstrap a *missing* `hash` field (it aborts on URL/hash count mismatch).

Hashes are lowercase hex by convention — Scoop's `format_hash` enforces `.toLower()`.

Debug with `scoop config debug $true`. Other checkver flags: `-f` (force), `-s` (skip updated), `-v VER` (use a given version).

This repo is auto-updated by the Excavator GitHub Action (`.github/workflows/excavator.yml`) on schedule and on push.

## Repo conventions (quick facts)

- 20/23 manifests use `shortcuts`; 6/23 use `bin` (most apps are GUI-only).
- `arm64` builds are rare — `bucket/zread.json` is the only manifest shipping an `arm64` variant.
- Chinese shortcut labels are allowed (e.g. `弹弹play`, `欧路词典`).
- `license` strings in use: `MIT`, `GPL-2.0`, `GPL-3.0`, `LGPL-3.0`, `Freeware`.
- URL-fragment rename (`#/dl.7z`, `#/dl.zip`) is the standard idiom to force an archive extension on a server that doesn't serve one.
- `Expand-7ZipArchive` is the helper to unwrap NSIS-style `*setup.exe` payloads.
- `checkver` + `autoupdate` should be on every manifest. A few in this repo currently ship only `version` — `bucket/at32-work-bench.json`, `bucket/ja-netfilter.json`, `bucket/logicanalyzer.json` — unfinished work awaiting a wired-up version source.
- In-repo reference: `bucket/zread.json` — GitHub releases, dual-arch `64bit`+`arm64` (no `32bit`), `#/` raw-`.exe` rename, `bin` shim, product homepage → `checkver` object form, `autoupdate` without a `hash` block (download fallback in CI).
- Reference examples — Main (`https://github.com/ScoopInstaller/Main/blob/master/bucket/<name>.json`): standard `architecture` + license object + shortcut subdirs → `7zip.json`; CLI single-arch + `env_*`/persist + `checkver:"github"` → `nvm.json`; dual-arch + per-arch `extract_dir` + `hash.url` → `nodejs.json`; `hash.find` (extract mode) → `curl.json`; `hash` via `$version` URL → `julia.json`; `hash` `rdf` mode → `imagemagick.json`; `checkver` capture groups (`$matchTag`) → `git.json`; `bin` alias shims → `busybox.json`; `suggest` → `ant.json`; `hash` `$url`-derivative → `apache.json`; `checkver:{github}` → `cmder.json`; `checkver` `jsonpath`/`jp` → `nuget.json`.
- Reference examples — Extras (`https://github.com/ScoopInstaller/Extras/blob/master/bucket/<name>.json`): GUI app + shortcuts + `Expand-7ZipArchive` unwrap → `heynote.json`; `Expand-7ZipArchive` dual-arch → `another-redis-desktop-manager.json`; shared download + per-arch `bin`/`shortcuts` + shortcut subdirs → `sysinternals.json`; SourceForge URL → `nsis.json`; `installer.script` (`Start-Process` installer exe) → `calibre.json`.

## References

- App Manifests: https://github.com/ScoopInstaller/Scoop/wiki/App-Manifests
- App Manifest Autoupdate: https://github.com/ScoopInstaller/Scoop/wiki/App-Manifest-Autoupdate
- Main bucket: https://github.com/ScoopInstaller/Main/tree/master/bucket
- Extras bucket: https://github.com/ScoopInstaller/Extras/tree/master/bucket
