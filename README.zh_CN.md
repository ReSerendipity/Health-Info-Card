<p align="right">
  <a href="README.md">English</a> · [简体中文](README.zh_CN.md)
</p>

# AI Passport · Senior Safety Card（健康信息卡）

**AI Passport Senior Safety Card（健康信息卡）**把 FoloToy AI Passport 变成一张**离线、可穿戴的求助卡**：老人迷路或需要帮助时，旁人不需要任何 App，就能从屏幕读到称呼、求助说明、家庭联系人、居住区域、健康提醒和微信二维码。

这是一个 **ESP-IDF 5.5.3 / ESP32-C3 嵌入式固件项目**：没有桌面 UI 可运行，交付物是烧录到设备的固件（Release 为 `.bin`）。

[![CI](https://github.com/ReSerendipity/Health-Info-Card/actions/workflows/ci.yml/badge.svg)](https://github.com/ReSerendipity/Health-Info-Card/actions/workflows/ci.yml)
[![Release](https://img.shields.io/github/v/tag/ReSerendipity/Health-Info-Card)](https://github.com/ReSerendipity/Health-Info-Card/releases)
[![License: MIT](https://img.shields.io/github/license/ReSerendipity/Health-Info-Card)](LICENSE)
[![Last commit](https://img.shields.io/github/last-commit/ReSerendipity/Health-Info-Card)](https://github.com/ReSerendipity/Health-Info-Card/commits/main)
[![Top language](https://img.shields.io/github/languages/top/ReSerendipity/Health-Info-Card)](https://github.com/ReSerendipity/Health-Info-Card)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen)](.github/CONTRIBUTING.md)

---

## 项目简介

Senior Safety Card（健康信息卡）是基于 FoloToy AI Passport（ESP32-C3 / 240×320 ST7789）的离线求助应用。设备以卡片形态随身佩戴，屏幕展示 **5 页安心资料**：

| 页 | 内容 |
| --- | --- |
| 1/5 | 姓名、年龄、血型、求助说明 |
| 2/5 | 家属联系人与电话（默认脱敏显示） |
| 3/5 | 带我回家：家庭住址（可选完整展示） |
| 4/5 | 健康提醒：过敏史、慢病、日常用药 |
| 5/5 | 微信二维码（最多 4 张：家属／备用／社区医生／其他） |

全部资料保存在设备本地（NVS 与 `imgstore` 分区），不依赖互联网服务。项目保留原版 AI Passport 的**小程序 BLE 安装契约**（受保护的 `cardid` 与永久 Recovery 分区），升级可沿用原有资料与二维码。

## 核心特性

- **完全离线**：资料只在设备 NVS、二维码只在 `imgstore`，从不上传；Wi-Fi 与本地设置页仅在设置期间运行，正常使用时关闭蓝牙。
- **5 页一键可读**：上／下键翻页；二维码页短按确认键全屏显示，按任意键返回；旁人无需任何 App 即可阅读。
- **低功耗**：展示页 60 秒无操作进入深度休眠，任意功能键唤醒；设置热点 5 分钟无操作休眠。
- **PIN 保护的重置**：长按确认键重新打开本地设置；管理密码只保存随机盐 + SHA-256 摘要，输入节流（5 次失败锁定 60 秒）抵御在线暴力破解。
- **隐私默认**：电话默认脱敏、完整住址可选展示；内置示例资料带红色"示例数据"横幅，保存真实资料后自动消失。
- **可自动验收**：USB-Serial-JTAG 截图 + 按键注入协议（FAP_SCREENSHOT_V1 / FAP_KEY_V1），`tools/fap_verify.py` 可自动完成 5 页真机验收。

## 展示

设备屏幕上的安心卡主界面（内置示例资料，带红色"示例数据"横幅）：

![安心卡主界面](https://gcore.jsdelivr.net/gh/ReSerendipity/Health-Info-Card@main/assets/images/senior-safety-card-cover.png)

设备正面（可穿戴卡片形态）：

![设备正面](https://gcore.jsdelivr.net/gh/ReSerendipity/Health-Info-Card@main/docs/assets/brand/ai-passport-front.png)

设备多角度视图：

![设备多视图](https://gcore.jsdelivr.net/gh/ReSerendipity/Health-Info-Card@main/docs/assets/brand/ai-passport-views.jpg)

设备与数据流架构（硬件、Flash 分区、本地设置 portal、隐私边界）：

![设备与数据流架构](https://gcore.jsdelivr.net/gh/ReSerendipity/Health-Info-Card@main/docs/assets/readme-health-info-card-architecture.svg)

卡片生成与日常使用流程（首次开机 → 本地设置 → 离线展示 → 维护与验收）：

![卡片生成与使用流程](https://gcore.jsdelivr.net/gh/ReSerendipity/Health-Info-Card@main/docs/assets/readme-health-info-card-flow.svg)

## 快速开始

**环境**：ESP-IDF 5.5.3（`IDF_PATH` 请使用正斜杠，官方 `recovery_boot_hook` 的 CMake 在反斜杠路径下会解析失败）、Python 3（含 `esptool`）。

**安装与构建**（仓库根目录）：

```bash
git clone https://github.com/ReSerendipity/Health-Info-Card.git
cd Health-Info-Card
source /path/to/esp-idf/export.sh        # ESP-IDF 5.5.3
idf.py build                             # 编译固件
```

**启动（烧录）**：发布与安装只使用合并后的完整镜像：

```bash
cd build
python -m esptool --chip esp32c3 merge_bin -o FoloToy-AI-Passport-full.bin "@..\build\flash_args"
python -m esptool --chip esp32c3 -p COM3 -b 460800 write_flash 0x0 FoloToy-AI-Passport-full.bin
```

已写入设备身份（provisioned）的机器，优先通过 AI Passport 小程序 Recovery 流程安装；**不要执行 `erase-flash`**，它会清除设备身份和永久 Recovery。

**最小验证**：仓库自带主机侧校验（无需真机）：

```bash
./tools/validate.sh --static
```

构建通过不等于真机验证。正式发布前，应在实体设备上按 [`docs/health-info-card-guide.md`](docs/health-info-card-guide.md) 检查首次设置、全部资料页、二维码扫码、管理密码、一分钟休眠、按键唤醒与 Recovery 入口，或用 `tools/fap_verify.py COM3 <out_dir>` 自动采集 5 页截图。

## 系统要求与兼容性

- ESP32-C3、8 MB Flash、3 MB 主应用分区（`partitions.csv`）。
- `cardid` 固定于 `0x356000`（受保护，设备身份）；永久 Recovery 固定于 `0x700000`；开机持续按住上键 5 秒进入 Recovery。
- NVS 命名空间、键名与 `imgstore`／`imgframe` 地址与原版一致：只要分区内容保留，升级后可沿用原有资料与二维码。
- 二维码图片单张上限 28 KB（`imgstore` 4 槽，合计约 112 KB）；前端将手机图片缩为 240×240 JPEG。
- 上游基线为 [`FoloToy/ai-passport`](https://github.com/FoloToy/ai-passport)，仓库通过 `sync-main.yml` 每日自动同步。

## 使用示例

日常使用（离线）：

1. 上／下键在 5 页间切换；二维码页短按确认键全屏显示，按任意键返回。
2. 长按确认键重新打开本地设置：扫描屏幕 Wi-Fi 二维码 → 连接热点 → 浏览器打开 `http://192.168.4.1` → 验证管理密码后修改资料。
3. 展示页 60 秒无操作进入深度休眠，任意功能键唤醒并回到休眠前页面（RTC 内存）。

真机自动验收（USB-Serial-JTAG）：

```bash
python tools/fap_verify.py COM3 _fap_shots    # 自动采集并翻页全部 5 页
```

## FAQ

1. **这是医疗器械吗？** 不是。它只是一张实体信息卡，不提供医疗建议，也不替代急救服务或专业照护。
2. **数据会被上传吗？** 不会。资料只存设备 NVS、二维码只存 `imgstore`；Wi-Fi 仅在设置期间运行，正常使用关闭蓝牙。
3. **如何重新设置资料？** 长按确认键打开本地设置；若已设置管理密码，设置页会要求验证（5 次失败锁定 60 秒）。
4. **为什么屏幕上一直有红色"示例数据"横幅？** 内置占位资料带有 `demo` 标记；从设置页保存真实资料后，横幅自动消失。
5. **如何升级固件？** 优先使用小程序 Recovery 流程安装新版本；分区内容保留时，资料与二维码可继续沿用。不要 `erase-flash`。

## 文档与治理

- 文档索引：[`docs/INDEX.md`](docs/INDEX.md)；权威变更记录：[`docs/CHANGELOG.md`](docs/CHANGELOG.md)。
- 交付指南（字段、占位数据替换、真机验收结果）：[`docs/health-info-card-guide.md`](docs/health-info-card-guide.md)。
- CI：`ci.yml`（PR 静态检查 + ESP-IDF 5.5.3 固件构建）· `build-firmware.yml`（tag 构建并发布 `.bin` + `SHA256SUMS.txt`）· `deploy-pages.yml`（docs → GitHub Pages）· `sync-main.yml`（每日同步上游）。
- AI 协作入口：`local/AGENTS.md`（本地文件，不入库）；开发流程见 [`docs/development/agent-guide.md`](docs/development/agent-guide.md)。

## 贡献 / 安全 / 支持

- 贡献指南：[`.github/CONTRIBUTING.md`](.github/CONTRIBUTING.md) · 行为准则：[`.github/CODE_OF_CONDUCT.md`](.github/CODE_OF_CONDUCT.md)
- 安全报告：[`.github/SECURITY.md`](.github/SECURITY.md) · 支持渠道：[`.github/SUPPORT.md`](.github/SUPPORT.md)

## 许可证

基于 [MIT License](LICENSE) 发布。版权所有 (c) 2026 FoloToy。
