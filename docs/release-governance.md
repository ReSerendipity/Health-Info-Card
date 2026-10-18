# 发布 / 回滚 规范（Health-Info-Card）

> **来源与权威**：公共条款见 `ReSerendipity/.github` 的 `REPOSITORY_GUIDELINES.md` §10（发布 / 分发统一规范）。
> 本文件是本仓的本地化发布治理文档，遵循上述模板，只填本仓特有内容。

---

## 0. 远端优先（根原则，不可改写）

发布链的构建、打包、扫描、上传、推送**全部在 GitHub Actions 远端完成**；本地只做三件事：改版本位、落 CHANGELOG、`git tag` + `git push`。本地不执行打包 / 镜像构建 / 发布上传。

## 1. 版本号规范

- 遵循 **SemVer** `MAJOR.MINOR.PATCH`；预发布用 `-alpha.N` / `-beta.N` / `-rc.N`。
- **唯一权威版本位**：根目录 `CMakeLists.txt` 中 `set(PROJECT_VER "<X.Y.Z>")`（当前 `1.0.0`）。
- 必须同步的其它版本位：无。
- 版本位漂移由 待定（负责人 + 触发条件） 钉住——漂一处即判红。
- 自动化方式：手动 tag（人工）。

## 2. CHANGELOG 管理

- 遵循 **Keep a Changelog**：`Added / Changed / Fixed / Deprecated / Removed / Security`。
- 每次合并 `main` 在 `[Unreleased]` 追加；type 映射 `feat→Added`、`fix→Fixed`、`refactor→Changed`。
- 发版时 `[Unreleased]` → `[VERSION] - YYYY-MM-DD`。
- 本仓 CHANGELOG 路径：`docs/CHANGELOG.md`（中英双语配对：`docs/CHANGELOG.md` / `docs/CHANGELOG.zh_CN.md`）。

## 3. 发布流程（远端）

1. 确认 `docs/CHANGELOG.md` 中 `[Unreleased]` 条目完整
2. 同步版本位：编辑根 `CMakeLists.txt` 中 `set(PROJECT_VER "<X.Y.Z>")`
3. 本地打 tag 并推送：`git tag v<X.Y.Z>-<app-name>` → `git push origin v<X.Y.Z>-<app-name>`
   - Tag 命名约定：`v<version>-<app-name>`（小写 kebab-case），例如 `v1.0.0-health-info-card`。
   - 同一源码树可能构建多个应用，tag 必须携带应用名以区分 Release。
4. 远端凭 tag 触发 `.github/workflows/build-firmware.yml` 构建固件产物
5. 远端自动创建 **Release** 并挂资产（`FoloToy-AI-Passport-full.bin` + `SHA256SUMS.txt`）
6. 人工审核后发布 Release 说明（What's new / How to build / How to use）
7. 盯到终态：`gh run list` 确认绿；红了当场修或 revert 止损

## 4. 分发产物与完整性

| 产物 | 构建位置 | 标签 / 命名 | 完整性手段 |
|---|---|---|---|
| `FoloToy-AI-Passport-full.bin`（ESP32-C3 固件合并镜像） | **远端** | `v<X.Y.Z>-<app-name>`（不可变 semver + 应用名；禁 floating latest） | `SHA256SUMS.txt` |

- 产物形态：ESP-IDF 5.5.3 编译的固件合并 bin（bootloader + partition table + application），从 `0x0` 烧录。
- 发布前验证：`tools/verify_firmware.py` 校验 bootloader 位置、分区表 MD5、应用偏移与 8 MB Flash 参数。
- **不使用 GPG 分离签名**（2026-10-18 裁决）。

### 发布产物判断依据

本仓**确有可分发产物**：ESP32-C3 固件合并镜像 `FoloToy-AI-Passport-full.bin`，用户通过浏览器 web flasher 烧录到硬件设备使用。因此已建立 tag 驱动的远端构建发布链（`.github/workflows/build-firmware.yml`），并补充 `SHA256SUMS.txt` 完整性校验。

## 5. 回滚

- **判定**（满足任一即触发）：待定（负责人 + 触发条件）
- **执行**：`git revert <bad-commit>`（保历史、可再 forward）；固件产物不可变，按上一版可用 tag 重新发布
- **破坏性动作**：重指向 / 删除 tag、移动任何远端 ref → **须仓库所有者单独授权**
- **回滚后验证**：`./tools/validate.sh --firmware` 构建并校验固件完整性

## 6. SLA / 错误预算

- 目标可用性 待定（负责人 + 触发条件）；超出即冻结非紧急发布并复盘。

## 7. 发布前检查清单

- [ ] 版本位已同步（根 `CMakeLists.txt` → `PROJECT_VER`）
- [ ] `docs/CHANGELOG.md` `[Unreleased]` 已改版本 + 日期
- [ ] `./tools/validate.sh --static` 通过（actionlint + 主机测试）
- [ ] `./tools/validate.sh --firmware` 构建校验通过
- [ ] **本地未产出任何发布交付物**（全部远端构建）
- [ ] Release 说明已撰写（What's new / How to build / How to use），与 CHANGELOG 一致

---

## 差异点采集清单

| 占位符 | 采集来源 | 本仓取值 |
|---|---|---|
| 版本位文件与字段 | 根 `CMakeLists.txt` → `set(PROJECT_VER "1.0.0")` | `CMakeLists.txt` `PROJECT_VER`（当前 1.0.0） |
| 其它需同步版本位 | `grep -rn` 版本串 | 无 |
| 版本一致性校验入口 | 无自动化版本一致性测试 | 待定（负责人 + 触发条件） |
| 自动化方式 | 无 release-please | 手动 tag（人工） |
| 发布 workflow 文件名 | `.github/workflows/build-firmware.yml` | `build-firmware.yml`（tag 触发构建 + Release） |
| 产物形态与标签规则 | 构建 workflow | `FoloToy-AI-Passport-full.bin`，tag = `v<X.Y.Z>-<app-name>` |
| 完整性手段 | Release 资产 | `SHA256SUMS.txt` |
| 回滚判定阈值 | 无运行手册 / 监控 | 待定（负责人 + 触发条件） |
| 回滚后验证命令 | `tools/validate.sh` | `./tools/validate.sh --firmware` |
| 技术栈 | ESP-IDF 5.5.3 / CMake | ESP32-C3 嵌入式固件 |
| 部署目标 | 浏览器 web flasher + GitHub Pages 文档 | 硬件设备烧录 + 文档站 |

## 禁止事项

- 不得重写 §10 的公共条款（远端优先、tag 驱动、版本位一致、不可变标签、无 GPG）。
- 不得引入 `latest` 作发布基线；不得保留或新建 GPG 签名流程。
- 不得留空占位符，不得编造阈值。
