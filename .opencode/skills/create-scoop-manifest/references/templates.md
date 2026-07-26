# Templates & canonical field order

The full JSON templates and the canonical key order for Scoop app manifests. Read this on every manifest creation — pick a template, copy it, fill in the app specifics, and confirm the key order matches.

## Contents

- [Canonical field order](#canonical-field-order)
- [Decision table](#decision-table)
- [Template A — single-arch, GitHub releases](#template-a--single-arch-github-releases)
- [Template B — dual-arch, per-arch extract_dir](#template-b--dual-arch-per-arch-extract_dir)
- [Minimal flat — no architecture block](#minimal-flat--no-architecture-block)
- [Template C — 64bit + arm64, raw .exe, product homepage](#template-c--64bit--arm64-raw-exe-product-homepage)

## Canonical field order

Aligned with ScoopInstaller/Main conventions. `bin/formatjson.ps1` normalizes whitespace and values but does not reorder keys, so match this order by hand — it keeps diffs reviewable.

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

## Decision table

| Scenario | Template |
|---|---|
| Single-arch, GitHub releases (common GUI app) | Template A |
| Dual-arch (64bit + 32bit), per-arch `extract_dir` | Template B |
| Dual-arch (64bit + arm64), raw `.exe`, product homepage, no checksum | Template C |
| No `architecture` block, flat single download | Minimal flat |

## Template A — single-arch, GitHub releases

Common GUI-app pattern; see Extras' `heynote.json`.

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

## Template B — dual-arch, per-arch extract_dir

Mirrors Main's `nodejs.json`.

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

## Minimal flat — no architecture block

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

## Template C — 64bit + arm64, raw .exe, product homepage

Mirrors `bucket/zread.json`. The `#/` fragment renames each per-arch binary to a stable `bin` target.

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
