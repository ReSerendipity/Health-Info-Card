<p align="right">
  <a href="CHANGELOG.zh_CN.md">简体中文</a> · <strong>English</strong>
</p>

# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.0.0] - 2026-10-09

### Added

- Initialize the FoloToy-Card firmware demo.
- Add the Senior Safety Card application.
- Show age and blood type on the health info card.
- Preserve mini-program BLE recovery.
- Add FAP serial screenshot and key-injection workflow.
- Add wireless and low-power demo examples.
- Add four-slot QR storage with paging-key browsing.
- Add safety card profile v3 with portal validation and HD QR upload.

### Fixed

- Declare `_POSIX_C_SOURCE` in `safety_profile` to fix the implicit `strnlen` declaration under CI `-std=c11`.
- Move the demo warning off the content panel so it no longer overlaps the profile title.
- Read-modify-write the imgstore header sector across slot operations.
- Normalize the IDF_PATH include dir for Windows builds.
- Isolate the firmware validation configuration in CI.

[1.0.0]: https://github.com/ReSerendipity/Health-Info-Card/releases/tag/v1.0.0
