---
name: create-scoop-manifest
description: Create a new Scoop app manifest at bucket/<name>.json in this bucket repo. Use whenever the user asks to add, scaffold, or draft a new app/package/tool manifest — e.g. "add <app> to the bucket", "create a manifest for <app>", "scoopify <repo>". Trigger even when the user does not say "manifest" or "scoop" explicitly. Do not use for the deprecated/ folder (move a manifest there only when retiring an app), and do not use to edit existing manifests unless explicitly asked; when a manifest is touched, reorder its keys to the canonical field order.
---

# Create a Scoop manifest

## Scope

Produce `bucket/<name>.json` — a complete, validated Scoop app manifest. The output ships with `version`, `checkver`, and `autoupdate` wired together; a version-only manifest is unfinished.

`bin/formatjson.ps1` normalizes whitespace and values but does not reorder keys, so reorder any manifest you touch to the canonical field order — it keeps diffs reviewable and matches ScoopInstaller/Main conventions. The field order and full templates live in `references/templates.md`.

## File placement & naming

- Path: `bucket/<name>.json`.
- Name: lowercase, hyphen-separated, matching the app's canonical id (e.g. `nat-type-tester`, `mpv.net-dw`). Dots are allowed when matching a real product name.
- Retiring an app: move `bucket/<name>.json` → `deprecated/<name>.json`.

## Templates

| Scenario | Template |
|---|---|
| Single-arch, GitHub releases (common GUI app) | Template A |
| Dual-arch (64bit + 32bit), per-arch `extract_dir` | Template B |
| Dual-arch (64bit + arm64), raw `.exe`, product homepage, no checksum | Template C |
| No `architecture` block, flat single download | Minimal flat |

Full JSON for each, plus the canonical field order, in `references/templates.md`. For fields beyond these shapes (lifecycle scripts, installer objects, persistence, env, psmodule), read `references/field-reference.md`.

## checkver

Two common forms:

- `"github"` — homepage = repo URL; matches `/releases/tag/(?:v|V)?([\d.]+)` and ignores pre-releases.
- `{ "github": "https://github.com/<owner>/<repo>" }` — when `homepage` is a product/marketing page rather than the repo (e.g. `https://z.ai/`); the string form would fail with `checkver expects the homepage to be a github repository`. See `bucket/zread.json`.

`checkver` and `autoupdate` always appear together; if the version can't be derived yet, leave a TODO. For the full form catalog (bare regex, `url`+`regex`, `jsonpath`/`jp`, `xpath`, `reverse`, `replace`, `useragent`, `script`), read `references/checkver-autoupdate.md`.

## autoupdate

`autoupdate` mirrors the manifest (field list in `references/checkver-autoupdate.md`).

## Validation

Run from repo root, in this order. Scripts delegate to the local Scoop install (`scoop prefix scoop`); Scoop must be installed.

1. `.\bin\formatjson.ps1`            — normalize formatting/values, confirm JSON parses.
2. `.\bin\checkver.ps1 <app>`        — detected version matches `version`.
3. `.\bin\checkurls.ps1 <app>`       — every download URL resolves.
4. `.\bin\checkhashes.ps1 <app>`     — only when the manifest ships `hash`; for hashless manifests it errors "URLS and hashes count mismatch" (expected — skip).

Hashes are lowercase hex — Scoop's `format_hash` enforces `.toLower()`.

Debug flags and the Excavator schedule live in `references/repo-conventions.md`.

## References

- `references/templates.md` — canonical field order + the 4 full JSON templates. Read on every manifest creation.
- `references/field-reference.md` — per-field detail (download, architecture, lifecycle scripts, shims & shortcuts, persistence & deps, other). Read when a manifest needs fields beyond the templates.
- `references/checkver-autoupdate.md` — full checkver form catalog, `autoupdate.hash` modes, substitution variables. Read when `checkver:"github"` / `{ "github": "<repo>" }` doesn't fit the upstream's release scheme.
- `references/repo-conventions.md` — quick facts, in-repo reference, Main/Extras example manifests, Excavator schedule. Read when choosing a template or mirroring a tricky pattern.
- App Manifests: https://github.com/ScoopInstaller/Scoop/wiki/App-Manifests
- App Manifest Autoupdate: https://github.com/ScoopInstaller/Scoop/wiki/App-Manifest-Autoupdate
- Main bucket: https://github.com/ScoopInstaller/Main/tree/master/bucket
- Extras bucket: https://github.com/ScoopInstaller/Extras/tree/master/bucket
