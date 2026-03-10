# AGENTS.md

This file is the main AI guidance entry point for this repository.

## Project Overview

This is a **firmware distribution and manifest management repository** for the `esp-gif-frame` project. It is NOT a firmware source code repo — it hosts manifests, a web installer, and compiled binaries for Seeed Studio XIAO ESP32 devices (ESP32-C3 and ESP32-S3) running GIF frame firmware.

The upstream firmware source lives at: <https://github.com/printminion/seeedstudio-xiao-TFT_eSPI_GifPlayer_With_Touch>

## Key Commands

```bash
# Regenerate manifests.json from manifest/ source files
python genManifest.py
```

## Repository Structure

| Path | Purpose |
| ---- | ------- |
| `manifest/` | Source manifest JSON files — one per channel (`release`, `development`) |
| `manifest_ext/` | Generated manifests with absolute GitHub Pages URLs (output of `genManifest.py`) |
| `manifests.json` | Consolidated index consumed by the web installer, grouped by channel |
| `genManifest.py` | Resolves relative paths to absolute GitHub Pages URLs and builds `manifests.json` |
| `index.html` | Web-based firmware installer using ESP Web Tools v10 |
| `firmware/printminion-esp-gif-frame/` | Compiled `.bin` artifacts (on `firmware` branch only) |
| `firmware/map/` | Linker map files `.map.gz` (on `firmware` branch only) |

## Branch Layout

- **`main`** — source manifests and web installer
- **`firmware`** — published to GitHub Pages; contains compiled binaries, generated manifests, and `manifests.json`
- **`gh_actions`** — GitHub Actions workflow (`fetch_deploy.yml`) that orchestrates the full pipeline

## CI/CD Pipeline (`gh_actions` branch)

Triggered by `workflow_dispatch`, `repository_dispatch`, or push to `gh_actions`. Three jobs run in sequence:

1. **`empty_branch`** — force-resets the `firmware` branch to a clean state
2. **`download`** — merges `main` into `firmware`, downloads build artifacts from upstream (`build_devel.yml` in `seeedstudio-xiao-TFT_eSPI_GifPlayer_With_Touch`), places binaries in `firmware/printminion-esp-gif-frame/` and maps in `firmware/map/`, then runs `genManifest.py`
3. **`deploy`** — publishes the `firmware` branch to GitHub Pages

> Note: Only development/CI builds are downloaded automatically. Release firmware download is currently commented out in the workflow.

## Architecture

The web installer (`index.html`) fetches `manifests.json` from GitHub Pages, populates a device dropdown grouped by channel (`release` / `development`), and uses [ESP Web Tools](https://esphome.github.io/esp-web-tools/) (WebSerial + Improv protocol) to flash the selected firmware over USB.

`genManifest.py` processes each `manifest/*.manifest.json` by resolving relative paths to absolute `https://printminion.github.io/esp-gif-frame-firmware/...` URLs, writes extended copies to `manifest_ext/`, and merges them into `manifests.json`.

Firmware binary naming: `seeed_xiao_esp32{c3|s3}.factory.bin`

## Manifest JSON Schema

```json
{
  "name": "printminion esp-gif-frame",
  "new_install_prompt_erase": true,
  "funding_url": "https://paypal.me/printminion",
  "new_install_improv_wait_time": 10,
  "builds": [
    {
      "chipFamily": "XIAO ESP32-C3",
      "improv": true,
      "parts": [{ "path": "<absolute-url>.bin", "offset": 0 }]
    },
    {
      "chipFamily": "XIAO ESP32-S3",
      "improv": true,
      "parts": [{ "path": "<absolute-url>.bin", "offset": 0 }]
    }
  ]
}
```

Supported `chipFamily` values: `XIAO ESP32-C3`, `XIAO ESP32-S3`.

`manifests.json` groups entries by channel key (`"release"`, `"development"`), each entry having `path`, `name`, and `chipFamilies[]`.

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
