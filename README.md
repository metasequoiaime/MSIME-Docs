# 水杉输入法文档

<!-- badges:start -->
[![CI](https://img.shields.io/github/actions/workflow/status/metasequoiaime/msime-docs/docs.yml?branch=main&label=CI)](https://github.com/metasequoiaime/msime-docs/actions/workflows/docs.yml)
[![CodeQL](https://img.shields.io/github/actions/workflow/status/metasequoiaime/msime-docs/codeql.yml?branch=main&label=CodeQL)](https://github.com/metasequoiaime/msime-docs/actions/workflows/codeql.yml)
[![License](https://img.shields.io/github/license/metasequoiaime/msime-docs)](LICENSE)
[![Stars](https://img.shields.io/github/stars/metasequoiaime/msime-docs?style=flat)](https://github.com/metasequoiaime/msime-docs/stargazers)
<!-- badges:end -->

本仓是用户指南、产品架构和跨仓开发维护说明的统一入口。项目介绍、社区政策与参与方向见[组织主页](https://github.com/metasequoiaime)及[组织仓库](https://github.com/metasequoiaime/.github)。产品可用性以各平台 Release 和指南为准。

## 用户文档

遇到方框、安装失败、快捷键冲突或候选异常？先查[常见问题 Q&A](guides/faq.md)。

| 平台 | 阅读入口 | 范围 |
| --- | --- | --- |
| Windows 10 / 11 | [Windows 使用指南](guides/windows.md) | 安装、设置、输入模式、词库导入导出、语音和排障；对应 [msime-windows](https://github.com/metasequoiaime/msime-windows/releases) 发布的版本 |
| macOS 12+ | [macOS 使用指南](guides/macos.md) · [语音输入](guides/macos-voice.md) | 安装、原生设置、快捷键、数据与卸载；在 [msime](https://github.com/metasequoiaime/msime/releases) 发布 |
| Linux / IBus | [Linux 使用指南](guides/linux.md) | 安装启用、设置、桌面工具、语音命令与排障；已发布安装包来自已归档的 msime-linux，msime 中同时提供 IBus 与 Fcitx5 入口的新宿主仍在开发，尚未发布 |
| iOS | [iOS 宿主说明](https://github.com/metasequoiaime/msime/blob/develop/platforms/ios/README.md) · [下载页](https://msime.app/download/) | 键盘扩展，通过 TestFlight 公开测试分发，尚未上架 App Store |
| Android | [Android 宿主说明](https://github.com/metasequoiaime/msime/blob/develop/platforms/android/README.md) | 开发中，尚未发布安装包 |
| HarmonyOS | [HarmonyOS 宿主说明](https://github.com/metasequoiaime/msime/blob/develop/platforms/harmony/README.md) | 开发中，尚未发布安装包 |
| 跨平台 | [从其他输入法迁移](guides/migration.md) | 搜狗、微软拼音、Rime 的用户词库怎么带过来；导入格式目前按 Windows 版核对 |
| 跨平台 | [无障碍现状](guides/accessibility.md) | 屏幕阅读器支持的实际状况与可参与的方向 |

Windows、macOS、macOS 语音与 Linux 指南及常见问题（含繁體译文）是官网对应正文的唯一来源。修改用户说明请在本仓提交，msime-web 通过固定的 Docs 子模块渲染；构建/API 说明仍由各代码仓库维护。指南按已提交源码核对，具体安装版本可能有所不同，请结合 Release 阅读。

仓库边界与数据来源见[公共仓库与平台架构](architecture/repositories.md)。

## 繁體中文

[Windows 使用指南](guides/zh-TW/windows.md) · [macOS 使用指南](guides/zh-TW/macos.md) · [macOS 語音](guides/zh-TW/macos-voice.md) · [Linux 使用指南](guides/zh-TW/linux.md) · [常見問題](guides/zh-TW/faq.md)。

繁體譯文與簡體原文均在本倉維護，網站使用固定 gitlink 渲染。`guides/zh-TW/sources.json` 記錄翻譯核對的來源提交與內容摘要；原文變更時需同步核對譯文。程式碼、檔名及引號內的應用程式選項保留原文，避免與實際介面不符。

## 开发与维护

| 说明 | 阅读入口 |
| --- | --- |
| 仓库职责、输入链路、数据来源与问题归属 | [公共仓库与平台架构](architecture/repositories.md) |
| 各产品如何固定引擎与词库 | [平台固定版本](architecture/platform-adoption.md) |
| 进程边界、不可信输入、产物信任链与已知薄弱处 | [安全模型与信任边界](architecture/security-model.md) |
| CI 检查、健康审计与排障 | [持续集成与自动维护](development/continuous-integration.md) |
| 过往实施、核对依据与界面示例 | [历史记录](archive/README.md) |

## 开源代码

代码与模块入口集中在[架构说明](architecture/repositories.md)，本页不维护第二份仓库清单。

## 手动构建

本仓只有文档。模块 API、依赖安装、构建与测试命令以各实现仓库的 README 和工作流为准；跨仓维护说明见[开发与维护](#开发与维护)。

## 截图

[历史界面示例](archive/interface-examples.md)已归档；当前使用方式请看对应平台指南。

## 贡献

文档修改遵循[本仓贡献指南](CONTRIBUTING.md)。通用贡献流程、安全策略、治理与招募统一在[组织仓库](https://github.com/metasequoiaime/.github)维护。

## 感谢

历史签名证书支持：[Certum](https://www.certum.eu/en/)。

## 许可协议

GPL-3.0.

<!-- star-history:start -->
## Star History

<a href="https://star-history.com/#metasequoiaime/msime-docs&Date">
  <img src="https://api.star-history.com/svg?repos=metasequoiaime/msime-docs&type=Date" alt="Star History Chart" width="600">
</a>
<!-- star-history:end -->
