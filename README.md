# pcs-compressor-updates

Update manifest for **Paramount Compressor Suite** (`PmCp`,
`com.paramount.compressorsuite`).

This repo exists to serve exactly one file, plus the release assets the plug-in points
people at:

    https://raw.githubusercontent.com/soundslikehoot/pcs-compressor-updates/main/updates.json

The plug-in GETs it on editor open (at most once per 24 h, background thread, silent on
failure) and may draw a corner banner. It never downloads or installs anything — clicking
the banner opens the download page in the user's normal browser.

## There is no `updates.json` here yet, and that is the intended state

The manifest is RSA-signed, and the key is not on any machine that could publish
automatically. Until one is committed here the GET 404s, the check does nothing, and
nobody banners. A build shipped in that state is not broken — the feature is compiled in
and inert, waiting for the file.

## Why this is separate from `pcs-updates` and `pcs-advanced-updates`

`updates.json` has a fixed two-channel shape (`beta` / `release`) describing ONE product,
and the compressor is on its own version line.

The deeper reason is the cache. All three products share one product folder —
`~/Library/Application Support/Paramount Console Suite/` — deliberately, so a single
licence folder serves all of them. `UpdateCheck::cacheFile()` is that folder plus the
compile-time cache name, so **the filename is the only thing keeping the three products'
manifests apart on disk**:

| product | manifest repo | cache file |
|---|---|---|
| Paramount Console Suite | `pcs-updates` | `update-cache.json` |
| Paramount Console Suite Advanced | `pcs-advanced-updates` | `update-cache-advanced.json` |
| Paramount Compressor Suite | `pcs-compressor-updates` | `update-cache-compressor.json` |

Point two products at one cache name and the second reads the first's cached manifest,
verifies it perfectly — one key signs all of them, and the manifest carries no product
field — and offers its user the wrong product's installer, with no network involved and
nothing in its own configuration looking wrong.

That is asserted in both directions now:
`Tools/verify_signing.sh --edition compressor` fails if this product's binary contains
either of the console's two strings, and its build markers require its own.

## Publishing

The manifest is RSA-signed with the same key as the licence files. Chain every new
manifest from whatever is LIVE rather than authoring from scratch — `--in` refuses input
that does not verify against the baked-in public key, and chaining carries untouched
channels forward so publishing a beta cannot blank the release channel.

Builds default to the `beta` channel, so a release means **both** channels or nobody
banners.

See `Tools/RELEASING.md` in the plug-in repo for the full ritual.
