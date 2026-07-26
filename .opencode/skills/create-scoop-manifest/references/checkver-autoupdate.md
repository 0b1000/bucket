# checkver & autoupdate reference

Full version-detection and auto-update reference. Read this when the common `checkver:"github"` / `{ "github": "<repo>" }` shapes in SKILL.md don't fit the upstream's release scheme — e.g. scraping a JSON endpoint, deriving a version from a captured regex group, or wiring `autoupdate.hash` to pull upstream checksums.

## Contents

- [checkver forms](#checkver-forms)
- [autoupdate](#autoupdate)
- [autoupdate.hash sub-property](#autoupdatehash-sub-property)
- [Substitution variables](#substitution-variables)

## checkver forms

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

## autoupdate.hash sub-property

`autoupdate.hash`:

- `mode`: `extract` (default; plain text/webpage via regex) | `json` | `xpath` | `rdf` | `metalink` | `fosshub` (auto) | `sourceforge` (auto) | `download` (fallback: download and hash locally).
- `url`: template for the hash source. Under `mode: extract`, built-in regexes apply: `^([a-fA-F0-9]+)$` and `([a-fA-F0-9]{32,128})[\x20\t]+.*$basename(?:[\x20\t]+\d+)?`.
- `regex` / `find`: custom regex (supports hash variables like `$sha256`).
- `jsonpath` / `jp`, `xpath`: for JSON / XML sources.

FossHub / SourceForge download URLs are auto-detected, so no `hash` block is needed.

**Upstream publishes a checksum?** Wire `autoupdate.hash` to pull it (a SHA256SUMS file, checksums page, or API field) — Scoop fetches the hash without downloading the binary. Fast.

**Upstream publishes no checksum?** Keep the manifest's `"hash": "<sha256>"` placeholder and run `.\bin\checkhashes.ps1 <app> -Update` — Scoop downloads each URL via its own fetcher and writes the real SHA256 back (the placeholder must be present; a missing `hash` field aborts with "URLS and hashes count mismatch"). Excavator's download fallback only computes a hash on a version bump (`Invoke-AutoUpdate`, for the new version's URL), never backfilling the already-shipped version. Do not hand-roll `Get-FileHash` + `Invoke-WebRequest`/`curl`.

## Substitution variables

- Version: `$version`, `$underscoreVersion`, `$dashVersion`, `$cleanVersion`, `$majorVersion` / `$minorVersion` / `$patchVersion` / `$buildVersion`, `$matchHead` (first 2–3 dotted segments), `$matchTail`, `$preReleaseVersion`.
- URL: `$url` (no fragment), `$baseurl` (no filename or fragment), `$basename` (filename, ignores `#/...`).
- Hash: `$md5`, `$sha1`, `$sha256`, `$sha512`, `$checksum` (any), `$base64`.
- Captured (from `checkver.regex` groups): use `$match1` / `$matchName` inside `autoupdate`; use `${1}` / `${name}` inside `checkver.replace`. Named-group variable names are camelCase with only the first letter uppercase (e.g. `$matchVersion`, `$matchShort`).
