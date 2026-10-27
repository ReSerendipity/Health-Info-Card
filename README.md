<p align="right">
  [English](README.md) · <a href="README.zh_CN.md">简体中文</a>
</p>

# AI Passport · Senior Safety Card (Health-Info-Card)

**AI Passport Senior Safety Card (Health-Info-Card)** turns the FoloToy AI Passport into an **offline, wearable help card**: when an older adult gets lost or needs help, a passer-by can read the wearer's name, help message, family contacts, approximate home area, health notes and WeChat QR codes directly from the screen — no app required.

This is an **ESP-IDF 5.5.3 / ESP32-C3 embedded firmware project**: there is no desktop UI to run; the deliverable is firmware flashed to the device (releases ship a `.bin`).

[![CI](https://github.com/ReSerendipity/Health-Info-Card/actions/workflows/ci.yml/badge.svg)](https://github.com/ReSerendipity/Health-Info-Card/actions/workflows/ci.yml)
[![Release](https://img.shields.io/github/v/tag/ReSerendipity/Health-Info-Card)](https://github.com/ReSerendipity/Health-Info-Card/releases)
[![License: MIT](https://img.shields.io/github/license/ReSerendipity/Health-Info-Card)](LICENSE)
[![Last commit](https://img.shields.io/github/last-commit/ReSerendipity/Health-Info-Card)](https://github.com/ReSerendipity/Health-Info-Card/commits/main)
[![Top language](https://img.shields.io/github/languages/top/ReSerendipity/Health-Info-Card)](https://github.com/ReSerendipity/Health-Info-Card)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen)](.github/CONTRIBUTING.md)
[![Governance](https://img.shields.io/badge/governance-ReSerendipity%2F.github-blue)](https://github.com/ReSerendipity/.github/tree/main/governance)

---

## Introduction

Senior Safety Card (Health-Info-Card) is an offline help application built on the FoloToy AI Passport (ESP32-C3 / 240×320 ST7789). Worn as a physical card, it shows **five pages of safety information**:

| Page | Content |
| --- | --- |
| 1/5 | Name, age, blood type, help request |
| 2/5 | Family contacts and phones (masked by default) |
| 3/5 | Take me home: home address (full display opt-in) |
| 4/5 | Health reminders: allergies, chronic conditions, daily medication |
| 5/5 | WeChat QR codes (up to 4: family / backup / community doctor / other) |

All data is stored on the device itself (NVS and the `imgstore` partition) and never depends on an internet service. The project keeps the original AI Passport **mini-program BLE install contract** (protected `cardid` and permanent Recovery partitions), so an existing profile and QR images can be reused on upgrade.

## Core features

- **Fully offline**: the profile lives in the device NVS partition and QR images in the `imgstore` partition — nothing is uploaded. Wi-Fi and the local setup page run only during setup; Bluetooth is disabled during normal use.
- **Five readable pages**: UP/DOWN switch pages; OK shows a QR code full screen (any key returns). Anyone can read the card without an app.
- **Low power**: the display deep-sleeps after 60 s without input; any function key wakes it. The setup hotspot sleeps after five minutes without input.
- **PIN-protected reconfiguration**: long-press OK to reopen local setup; the PIN is stored only as a random salt plus SHA-256 digest, and attempts are throttled (5 failures lock for 60 s) against brute force.
- **Privacy by default**: phone numbers are masked by default and full address display is opt-in; the built-in placeholder profile shows a red "sample data" banner until a real profile is saved.
- **Automated acceptance**: an embedded screenshot/key-injection protocol over USB-Serial-JTAG (FAP_SCREENSHOT_V1 / FAP_KEY_V1) lets `tools/fap_verify.py` capture all five pages on real hardware.

## Screenshots

The safety-card main screen (built-in sample profile with the red "sample data" banner):

![Safety card main screen](https://gcore.jsdelivr.net/gh/ReSerendipity/Health-Info-Card@main/assets/images/senior-safety-card-cover.png)

Device front (wearable card form factor):

![Device front](https://gcore.jsdelivr.net/gh/ReSerendipity/Health-Info-Card@main/docs/assets/brand/ai-passport-front.png)

Device multi-angle views:

![Device views](https://gcore.jsdelivr.net/gh/ReSerendipity/Health-Info-Card@main/docs/assets/brand/ai-passport-views.jpg)

Device and data-flow architecture (hardware, Flash partitions, local setup portal, privacy boundary):

![Device and data-flow architecture](https://gcore.jsdelivr.net/gh/ReSerendipity/Health-Info-Card@main/docs/assets/readme-health-info-card-architecture.svg)

Setup and daily-use flow (first boot → local setup → offline display → maintenance and verification):

![Setup and daily-use flow](https://gcore.jsdelivr.net/gh/ReSerendipity/Health-Info-Card@main/docs/assets/readme-health-info-card-flow.svg)

## Quick start

**Environment**: ESP-IDF 5.5.3 (`IDF_PATH` must use forward slashes — the official `recovery_boot_hook` CMake breaks on backslash paths), Python 3 (with `esptool`).

**Build** (repository root):

```bash
git clone https://github.com/ReSerendipity/Health-Info-Card.git
cd Health-Info-Card
source /path/to/esp-idf/export.sh        # ESP-IDF 5.5.3
idf.py build
```

**Flash**: publish and install only the merged full image:

```bash
cd build
python -m esptool --chip esp32c3 merge_bin -o FoloToy-AI-Passport-full.bin "@..\build\flash_args"
python -m esptool --chip esp32c3 -p COM3 -b 460800 write_flash 0x0 FoloToy-AI-Passport-full.bin
```

On a provisioned device, prefer the AI Passport mini-program Recovery flow. Never use `erase-flash`; it destroys per-device identity and the permanent Recovery.

**Minimal validation** (host-side, no hardware required):

```bash
./tools/validate.sh --static
```

Build success is not device validation. Before treating a release as hardware-verified, walk the first-setup flow, all pages, QR scanning, PIN protection, one-minute sleep, key wake and Recovery entry on physical hardware per [`docs/health-info-card-guide.md`](docs/health-info-card-guide.md), or run `tools/fap_verify.py COM3 <out_dir>` to capture all five pages automatically.

## System requirements and compatibility

- ESP32-C3 with 8 MB Flash and a 3 MB application partition (`partitions.csv`).
- Protected `cardid` at `0x356000` (device identity); permanent Recovery at `0x700000`; hold UP for five seconds at boot to enter Recovery.
- NVS namespace/keys and `imgstore`/`imgframe` addresses match the original release, so an existing profile and QR images can be reused while the partition contents are preserved.
- QR images: 28 KB max each (`imgstore` 4 slots, ~112 KB total); the portal downscales phone images to 240×240 JPEG.
- Upstream baseline: [`FoloToy/ai-passport`](https://github.com/FoloToy/ai-passport), synchronized daily via `sync-main.yml`.

## Usage examples

Daily use (offline):

1. UP/DOWN page through the five pages; on a QR page, press OK for full screen; press any key to return.
2. Long-press OK to reopen local setup: scan the on-screen Wi-Fi QR → join the hotspot → open `http://192.168.4.1` in a browser → verify the PIN and edit the profile.
3. After 60 s without input the display deep-sleeps; any function key wakes it and returns to the previous page (RTC memory).

Automatic on-device acceptance (USB-Serial-JTAG):

```bash
python tools/fap_verify.py COM3 _fap_shots    # captures and pages through all five pages
```

## FAQ

1. **Is this a medical device?** No. It is a physical information card: it gives no medical advice and does not replace emergency services or professional care.
2. **Is my data uploaded anywhere?** No. The profile stays in NVS and QR images stay in `imgstore`; Wi-Fi runs only during setup and Bluetooth is disabled during normal use.
3. **How do I reconfigure the profile?** Long-press OK to open local setup; a configured PIN is required (5 failures lock the portal for 60 s).
4. **Why is there a red "sample data" banner on every page?** The built-in placeholder profile carries a `demo` flag; saving a real profile from the portal clears it.
5. **How do I upgrade the firmware?** Prefer the mini-program Recovery flow; existing profile and QR images are reused while partitions are preserved. Never `erase-flash`.

## Documentation and governance

- Documentation index: [`docs/INDEX.md`](docs/INDEX.md); authoritative changelog: [`docs/CHANGELOG.md`](docs/CHANGELOG.md).
- Delivery guide (fields, placeholder-data replacement, on-device acceptance results): [`docs/health-info-card-guide.md`](docs/health-info-card-guide.md).
- CI: `ci.yml` (PR checks + ESP-IDF 5.5.3 firmware build) · `build-firmware.yml` (tagged builds publish `.bin` + `SHA256SUMS.txt`) · `deploy-pages.yml` (docs → GitHub Pages) · `sync-main.yml` (daily upstream sync).
- AI-assisted development: `local/AGENTS.md` (local-only, not tracked); workflow in [`docs/development/agent-guide.md`](docs/development/agent-guide.md).

## Contributing / security / support

- Contributing: [`.github/CONTRIBUTING.md`](.github/CONTRIBUTING.md) · Code of conduct: [`.github/CODE_OF_CONDUCT.md`](.github/CODE_OF_CONDUCT.md)
- Security: [`.github/SECURITY.md`](.github/SECURITY.md) · Support: [`.github/SUPPORT.md`](.github/SUPPORT.md)

## License

Released under the [MIT License](LICENSE). Copyright (c) 2026 FoloToy.
