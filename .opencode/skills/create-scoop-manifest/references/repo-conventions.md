# Repo conventions

Quick facts and reference-example manifests for this bucket. Read this when choosing a template, deciding on `arm64`/shortcuts/bin, or looking for a Main/Extras manifest to mirror for a tricky pattern.

## Contents

- [Quick facts](#quick-facts)
- [In-repo reference](#in-repo-reference)
- [Reference examples — Main](#reference-examples--main)
- [Reference examples — Extras](#reference-examples--extras)
- [Operations](#operations)

## Quick facts

- 20/23 manifests use `shortcuts`; 6/23 use `bin` (most apps are GUI-only).
- `arm64` builds are rare — `bucket/zread.json` is the only manifest shipping an `arm64` variant.
- Chinese shortcut labels are allowed (e.g. `弹弹play`, `欧路词典`).
- `license` strings in use: `MIT`, `GPL-2.0`, `GPL-3.0`, `LGPL-3.0`, `Freeware`, `Proprietary`.
- URL-fragment rename (`#/dl.7z`, `#/dl.zip`) is the standard idiom to force an archive extension on a server that doesn't serve one.
- `Expand-7ZipArchive` is the helper to unwrap NSIS-style `*setup.exe` payloads.
- `checkver` + `autoupdate` should be on every manifest. A few in this repo currently ship only `version` — `bucket/at32-work-bench.json`, `bucket/ja-netfilter.json`, `bucket/logicanalyzer.json` — unfinished work awaiting a wired-up version source.

## In-repo reference

`bucket/zread.json` — GitHub releases, dual-arch `64bit`+`arm64` (no `32bit`), `#/` raw-`.exe` rename, `bin` shim, product homepage → `checkver` object form, `autoupdate` without a `hash` block (download fallback in CI).

## Reference examples — Main

`https://github.com/ScoopInstaller/Main/blob/master/bucket/<name>.json`:

- standard `architecture` + license object + shortcut subdirs → `7zip.json`
- CLI single-arch + `env_*`/persist + `checkver:"github"` → `nvm.json`
- dual-arch + per-arch `extract_dir` + `hash.url` → `nodejs.json`
- `hash.find` (extract mode) → `curl.json`
- `hash` via `$version` URL → `julia.json`
- `hash` `rdf` mode → `imagemagick.json`
- `checkver` capture groups (`$matchTag`) → `git.json`
- `bin` alias shims → `busybox.json`
- `suggest` → `ant.json`
- `hash` `$url`-derivative → `apache.json`
- `checkver:{github}` → `cmder.json`
- `checkver` `jsonpath`/`jp` → `nuget.json`

## Reference examples — Extras

`https://github.com/ScoopInstaller/Extras/blob/master/bucket/<name>.json`:

- GUI app + shortcuts + `Expand-7ZipArchive` unwrap → `heynote.json`
- `Expand-7ZipArchive` dual-arch → `another-redis-desktop-manager.json`
- shared download + per-arch `bin`/`shortcuts` + shortcut subdirs → `sysinternals.json`
- SourceForge URL → `nsis.json`
- `installer.script` (`Start-Process` installer exe) → `calibre.json`

## Operations

- This repo is auto-updated by the Excavator GitHub Action (`.github/workflows/excavator.yml`) on schedule and on push.
- Debug with `scoop config debug $true`. Other checkver flags: `-f` (force), `-s` (skip updated), `-v VER` (use a given version).
