# 架构 — Health-Info-Card

> 治理定位：`delivery=release`、`lifecycle=maintenance`（`governance/desired-state.json`）。
> 本文按 2026-11-03 的实际目录结构生成；数据流与边界须由维护者在实现变更时同步复核。

## 1. 定位

[English](README.md) · <a href="README.zh_CN.md">简体中文</a>

## 2. 组件与依赖方向

```text
应用层：components/ (14)
工具层：tools/ (8)
资产层：assets/ (9)
测试层：tests/ (5)
支撑层：docs/ (80)、.github/ (19)
其他层：main/ (36)、plays/ (10)、skills/ (8)、bootloader_components/ (2)
根级层：根级文件 (12)
```

| 组件 | 目录 | 文件数 | 代表文件 |
|---|---|---|---|
| `docs` | `docs/` | 80 | `docs/CHANGELOG.md`, `docs/CHANGELOG.zh_CN.md`, `docs/INDEX.md` |
| `main` | `main/` | 36 | `main/CMakeLists.txt`, `main/demo.h`, `main/demo_audio.c` |
| `components` | `components/` | 14 | `components/bsp/CMakeLists.txt`, `components/bsp/idf_component.yml`, `components/bsp/include/bsp_audio.h` |
| 根级文件 | `/` | 12 | `.clangd`, `.editorconfig`, `.gitignore` |
| `plays` | `plays/` | 10 | `plays/.gitkeep`, `plays/INDEX.md`, `plays/INDEX.zh_CN.md` |
| `assets` | `assets/` | 9 | `assets/README.md`, `assets/README.zh_CN.md`, `assets/fonts/README.md` |
| `skills` | `skills/` | 8 | `skills/README.md`, `skills/README.zh_CN.md`, `skills/experience-pr/SKILL.md` |
| `tools` | `tools/` | 8 | `tools/check_repo.py`, `tools/fap_device.py`, `tools/fap_verify.py` |
| `tests` | `tests/` | 5 | `tests/test_pin_throttle.c`, `tests/test_safety_contract.py`, `tests/test_safety_profile.c` |
| `bootloader_components` | `bootloader_components/` | 2 | `bootloader_components/recovery_boot_hook/CMakeLists.txt`, `bootloader_components/recovery_boot_hook/recovery_boot_hook.c` |

依赖方向：入口层（应用 / 页面）调用领域层（业务模块），领域层依赖基础设施层（脚本、配置、模型资产）；反向引用不允许。

## 3. 数据流

VitePress 读取 Markdown 数据 → 构建静态页 → 部署到 Pages。

## 4. 外部依赖

| 依赖 | 用途 | 不可用时 |
|---|---|---|
| Node.js / npm | VitePress 构建 | 无法生成站点 |

## 5. 边界与非目标

- 发布链只走 §10：远端优先、tag 驱动、资产附 `SHA256SUMS.txt`。
- 分发产物由 CI 生成，不入库。
- 不入库：密钥 / token / 口令 / 连接串、本机绝对路径、`local/` 与 `AGENTS.md`、构建产物。
- 治理正文只在 `ReSerendipity/.github/governance`，本仓不复制规则正文。

## 6. 变更影响面

| 改什么 | 必须同步 | 由谁校验 |
|---|---|---|
| 目录结构 / 入口 | 本文「组件与依赖方向」与「数据流」 | 维护者 |
| CI 与门禁 | 本仓 `README.md` 的测试命令；治理仓 §20 远端载体口径 | `audit-drift.py` |
| 依赖清单 | 锁文件与 `.github/dependabot.yml` 的生态段 | Dependabot / `lock-consistency` |
| README 展示图 | §8.1 真实性红线（图必须来自真实运行） | 治理仓 `readme-image-provenance` |
