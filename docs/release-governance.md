<p align="right">
  <a href="release-governance.zh_CN.md">简体中文</a> · <strong>English</strong>
</p>

# Release & Rollback Governance (Health-Info-Card)

> **Source of authority**: Common clauses are in `ReSerendipity/.github` → `REPOSITORY_GUIDELINES.md` §10 (Release / Package / Deployment standard).
> This file is the repo-local governance document, filling the template with repo-specific truths.

---

## 0. Remote-first — root principle, non-negotiable

The release chain — build, packaging, scanning, upload, push — **all happens on GitHub Actions runners in the cloud**. Local work is limited to three light actions: bump the version, update CHANGELOG, and `git tag` + `git push`. No local packaging, image builds, or release uploads.

## 1. Versioning

- Follows **SemVer** `MAJOR.MINOR.PATCH`; pre-releases use `-alpha.N` / `-beta.N` / `-rc.N`.
- **Single authoritative version site**: root `CMakeLists.txt` → `set(PROJECT_VER "<X.Y.Z>")` (currently `1.0.0`).
- Other version sites to sync: none.
- Version drift enforcement: TBD (owner + trigger).
- Automation: manual tag (human-driven).

## 2. CHANGELOG management

- Follows **Keep a Changelog**: `Added / Changed / Fixed / Deprecated / Removed / Security`.
- Append to `[Unreleased]` on every merge to `main`; map `feat→Added`, `fix→Fixed`, `refactor→Changed`.
- On release: `[Unreleased]` → `[VERSION] - YYYY-MM-DD`.
- Changelog path: `docs/CHANGELOG.md` (bilingual pair: `docs/CHANGELOG.md` / `docs/CHANGELOG.zh_CN.md`).

## 3. Release flow (remote)

1. Confirm `[Unreleased]` entries in `docs/CHANGELOG.md` are complete
2. Sync version: edit root `CMakeLists.txt` → `set(PROJECT_VER "<X.Y.Z>")`
3. Tag and push locally: `git tag v<X.Y.Z>-<app-name>` → `git push origin v<X.Y.Z>-<app-name>`
   - Tag naming: `v<version>-<app-name>` (lowercase kebab-case), e.g. `v1.0.0-health-info-card`.
   - When one source tree builds multiple apps, the tag must carry the app name to disambiguate releases.
4. The tag triggers `.github/workflows/build-firmware.yml` on the remote to build firmware
5. The remote automatically creates a **Release** with assets (`FoloToy-AI-Passport-full.bin` + `SHA256SUMS.txt`)
6. Manually draft release notes (What's new / How to build / How to use)
7. Watch to terminal state: `gh run list` must show green; on red, fix immediately or revert to stop the bleed

## 4. Distributable artifacts & integrity

| Artifact | Build location | Tag / naming | Integrity |
|---|---|---|---|
| `FoloToy-AI-Passport-full.bin` (ESP32-C3 merged firmware image) | **Remote** | `v<X.Y.Z>-<app-name>` (immutable semver + app name; no floating `latest`) | `SHA256SUMS.txt` |

- Artifact form: ESP-IDF 5.5.3 compiled merged binary (bootloader + partition table + application), flashed from `0x0`.
- Pre-release verification: `tools/verify_firmware.py` checks bootloader offset, partition table MD5, application offset, and 8 MB flash parameters.
- **No GPG detached signatures** (ruled out 2026-10-18).

### Artifact presence rationale

This repo **has distributable artifacts**: the ESP32-C3 merged firmware binary `FoloToy-AI-Passport-full.bin`, which users flash to hardware devices via a browser web flasher. The tag-driven remote build+release chain already exists as `.github/workflows/build-firmware.yml`; this governance doc adds the `SHA256SUMS.txt` integrity step per §10.1.

## 5. Rollback

- **Trigger** (any one): TBD (owner + trigger).
- **Execution**: `git revert <bad-commit>` (preserves history, can forward-fix later); firmware artifacts are immutable — re-release from the previous known-good tag.
- **Destructive actions**: retargeting / deleting tags, moving any remote ref → **requires explicit repo owner authorization**.
- **Post-rollback verification**: `./tools/validate.sh --firmware` builds and verifies firmware integrity.

## 6. SLA / error budget

- Target availability: TBD (owner + trigger). On breach, freeze non-urgent releases and run a postmortem.

## 7. Pre-release checklist

- [ ] Version site synced (root `CMakeLists.txt` → `PROJECT_VER`)
- [ ] `docs/CHANGELOG.md` `[Unreleased]` updated with version + date
- [ ] `./tools/validate.sh --static` passes (actionlint + host tests)
- [ ] `./tools/validate.sh --firmware` build & verification passes
- [ ] **No release artifacts produced locally** (all remote-built)
- [ ] Release notes drafted (What's new / How to build / How to use), consistent with CHANGELOG

---

## Difference-point collection checklist

| Placeholder | Source | Repo value |
|---|---|---|
| Version site file & field | Root `CMakeLists.txt` → `set(PROJECT_VER "1.0.0")` | `CMakeLists.txt` `PROJECT_VER` (currently 1.0.0) |
| Other version sites | `grep -rn` version strings | none |
| Version consistency check entry | No automated version-consistency test | TBD (owner + trigger) |
| Automation method | No release-please | Manual tag (human) |
| Release workflow filename | `.github/workflows/build-firmware.yml` | `build-firmware.yml` (tag-triggered build + Release) |
| Artifact form & tag rule | Build workflow | `FoloToy-AI-Passport-full.bin`, tag = `v<X.Y.Z>-<app-name>` |
| Integrity means | Release assets | `SHA256SUMS.txt` |
| Rollback trigger threshold | No runbook / monitoring | TBD (owner + trigger) |
| Post-rollback verify command | `tools/validate.sh` | `./tools/validate.sh --firmware` |
| Tech stack | ESP-IDF 5.5.3 / CMake | ESP32-C3 embedded firmware |
| Deployment target | Browser web flasher + GitHub Pages docs | Hardware device flashing + docs site |

## Prohibited

- Do not rewrite §10 common clauses (remote-first, tag-driven, version consistency, immutable tags, no GPG).
- Do not introduce `latest` as a release baseline; do not retain or create GPG signing flows.
- Do not leave placeholders blank; do not fabricate thresholds.