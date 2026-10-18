<p align="right">
  <strong>简体中文</strong> · <a href="CHANGELOG.md">English</a>
</p>

# Changelog

本文件记录本项目所有值得注意的变更。

格式基于 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/)，
版本号遵循[语义化版本](https://semver.org/lang/zh-CN/)（Semantic Versioning）。

## [Unreleased]

## [1.0.0] - 2026-10-09

### 新增（Added）

- 初始化 FoloToy-Card 固件示例。
- 新增老人安心卡（Senior Safety Card）应用。
- 健康信息卡上显示年龄与血型。
- 保留小程序 BLE 恢复能力。
- 新增 FAP 串口截图与按键注入工作流。
- 新增无线与低功耗示例。
- 新增四槽二维码存储与翻页浏览。
- 新增安心卡 profile v3，支持门户校验与高清二维码上传。

### 修复（Fixed）

- `safety_profile` 声明 `_POSIX_C_SOURCE`，修复 CI `-std=c11` 下 `strnlen` 隐式声明问题。
- 演示警告移出内容面板，不再遮挡资料标题。
- imgstore 跨槽操作采用读-改-写方式处理头部扇区。
- 归一化 Windows 构建的 IDF_PATH 包含目录。
- CI 中隔离固件校验配置。

[1.0.0]: https://github.com/ReSerendipity/Health-Info-Card/releases/tag/v1.0.0
