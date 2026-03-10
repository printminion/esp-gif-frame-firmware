# AGENTS.md

This file is the main AI guidance entry point for this repository.

## Project Overview

This is a **firmware distribution and manifest management repository** for the `esp-gif-frame` project. It is NOT a firmware source code repo — it hosts manifests, a web installer, and compiled binaries for Seeed Studio XIAO ESP32 devices (ESP32-C3 and ESP32-S3) running GIF frame firmware.

The upstream firmware source lives at: https://github.com/printminion/seeedstudio-xiao-TFT_eSPI_GifPlayer_With_Touch

## Key Commands

```bash
# Regenerate manifests.json from manifest/ source files
python genManifest.py
```

## Repository Structure

| Path | Purpose |
|------|---------|
| `manifest/` | Source manifest JSON files (one per device/channel) |
| `manifest_ext/` | Generated manifests with absolute GitHub Pages URLs (output of `genManifest.py`) |
| `genManifest.py` | Converts relative paths in manifests to absolute GitHub Pages URLs and generates `manifests.json` |
| `index.html` | Web-based firmware installer using ESP Web Tools v10 |

## Branch Layout

- **`main`** — distribution manifests and web installer
- **`firmware`** — compiled `.bin` artifacts published to GitHub Pages
- **`gh_actions`** — GitHub Actions workflow that pulls upstream builds, runs `genManifest.py`, and deploys to GitHub Pages

## Architecture

The web installer (`index.html`) fetches `manifests.json` from GitHub Pages, populates a device dropdown, and uses the [ESP Web Tools](https://esphome.github.io/esp-web-tools/) library (WebSerial + Improv protocol) to flash selected firmware over USB.

`genManifest.py` processes each `manifest/*.manifest.json` by resolving relative paths to absolute `https://printminion.github.io/esp-gif-frame-firmware/...` URLs, writes extended copies to `manifest_ext/`, and merges them into a single `manifests.json`.

Firmware binary naming convention: `app_seeed_xiao_esp32{c3|s3}-seeed_touch_104030087.bin`

## Manifest JSON Schema

```json
{
  "name": "Human-readable device name",
  "new_install_prompt_erase": true,
  "funding_url": "...",
  "new_install_improv_wait_time": 10,
  "builds": [
    {
      "chipFamily": "ESP32-C3",
      "improv": true,
      "parts": [{ "path": "<absolute-url>.bin", "offset": 0 }]
    }
  ]
}
```

Supported `chipFamily` values: `ESP32-C3`, `ESP32-S3`.

## Commit Messages

Use [Conventional Commits](https://www.conventionalcommits.org/):

```text
<type>(<scope>): <subject>
```

Common types: `feat`, `fix`, `chore`, `docs`, `refactor`, `ci`

Examples:

```text
feat(manifest): add ESP32-S3 development variant
fix(genManifest): handle missing offset field
ci(gh_actions): update firmware download step
docs: update installer instructions in index.html
```

**Rules:**

- Make atomic commits — one logical change per commit
- Write meaningful messages that explain *why*, not just *what*
- Do not add `Co-Authored-by` or any AI attribution lines to commit messages
