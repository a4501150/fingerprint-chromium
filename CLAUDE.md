# fingerprint-chromium

## Repo Structure

This is a fork of [adryfish/fingerprint-chromium](https://github.com/adryfish/fingerprint-chromium), which itself is a fork of [ungoogled-chromium](https://github.com/ungoogled-software/ungoogled-chromium) with fingerprint spoofing patches added.

Unlike the original upstream (which kept docs-only on main and source on version branches), we keep everything on **main** with one commit per Chromium version update: `"Update to Chromium X.X.X.X"`.

## Upstream Relationship

- **origin**: `git@github.com:a4501150/fingerprint-chromium.git` (our fork)
- **upstream**: `https://github.com/adryfish/fingerprint-chromium.git` (adryfish's original)
- **ungoogled-chromium**: `https://github.com/ungoogled-software/ungoogled-chromium.git` (the base project)

## How Version Upgrades Work

To upgrade to a new Chromium version (e.g., from 148 to 150):

1. Start from the matching ungoogled-chromium tag (e.g., `150.0.7871.100-1`) as the base
2. Port the 16 fingerprint patches from `patches/extra/fingerprint/`
3. Validate patches: `./devutils/validate_patches.py -r` (remote, no source download needed)
4. Or validate locally: download Chromium source + `patch --dry-run`
5. Squash into a single commit: `"Update to Chromium X.X.X.X"`

Key files that change between versions:
- `chromium_version.txt` — from the ungoogled-chromium base
- `patches/series` — add fingerprint entries at the end
- `patches/extra/fingerprint/*.patch` — may need context/line-number updates
- `flags.gn` — from the ungoogled-chromium base (don't modify)

## Fingerprint Patches (16 total)

Located in `patches/extra/fingerprint/`, applied after all ungoogled-chromium patches:

| Patch | What it does | Risk on upgrade |
|-------|-------------|-----------------|
| `000-add-fingerprint-switches` | Adds CLI switches (`--fingerprint`, `--timezone`, etc.) | Low |
| `001-disable-runtime.enable` | Disables CDP Runtime.enable detection | Medium (V8 internals) |
| `002-user-agent-fingerprint` | UA/Client Hints spoofing | High (API changes often) |
| `003-audio-fingerprint` | WebAudio sample rate noise | Low-Medium |
| `005-hardware-concurrency` | CPU cores + device memory spoofing | Low |
| `006-font-fingerprint` | Font availability spoofing | High (font stack changes) |
| `007-shadow-root` | Exposes closed Shadow DOM via `fakeShadowRoot` | Low-Medium |
| `009-webdriver` | Hides `navigator.webdriver` | Low |
| `010-headless` | Changes HeadlessChrome to Chrome in UA | Low |
| `011-gpu-info` | WebGL vendor/renderer spoofing | Medium |
| `012-canvas-get-image-data` | Canvas pixel noise (getImageData) | High (Skia changes) |
| `013-canvas-toDataURL` | Canvas pixel noise (toDataURL) | High (PaintImage changes) |
| `014-client-rects` | ClientRect offset noise | Medium |
| `015-canvas-measure-text` | measureText noise | Medium |
| `016-webgl-readPixels` | WebGL readPixels noise | Medium |
| `018-timezone` | Timezone spoofing via `--timezone` | Medium |

## Building

This repo is consumed as a git submodule by platform-specific build repos:
- macOS: clone `ungoogled-chromium-macos`, point submodule here
- Linux: clone `ungoogled-chromium-portablelinux`, point submodule here  
- Windows: clone `ungoogled-chromium-windows`, point submodule here

Local macOS build setup is at `/Users/jinyangli/src/chromium-build/`.

## Absorbing Upstream Patches

When adryfish releases a new version with source code:
1. Check the version branch (e.g., `upstream/148.0.7778.215`)
2. Compare their `patches/extra/fingerprint/` with ours
3. Cherry-pick any new patches or improvements
4. Our patches were originally ported from the `144.0.7559.132` branch

When ungoogled-chromium releases a new version:
1. Use the new tag as the base (all `core/` and `extra/` non-fingerprint patches come from there)
2. Only port the `patches/extra/fingerprint/` directory and `patches/series` entries
3. The `bromite` patches are maintained by ungoogled-chromium — use theirs as-is

## Gotchas

- The `bromite/flag-fingerprinting-canvas-image-data-noise.patch` uses `base::RandIntInclusive` (changed from `base::RandInt` in Chrome 150+)
- The `components/ungoogled/` infrastructure (switches, BUILD.gn) is identical between ungoogled-chromium versions — fingerprint switch patches rarely need changes
- macOS platform patches (`ungoogled-chromium-macos`) may need context fixes even when targeting the same Chrome version — verify with `patch --dry-run`
- `fix-nasm-build-config.patch` was added in Chrome 150 macOS repo, not present in 148
