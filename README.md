# 水杉输入法文档

本仓是用户指南、产品架构和跨仓开发维护说明的统一入口。项目介绍、社区政策与参与方向见[组织主页](https://github.com/metasequoiaime)及[组织仓库](https://github.com/metasequoiaime/.github)。产品可用性以各平台 Release 和指南为准。

## 用户文档

| 平台 | 阅读入口 | 范围 |
| --- | --- | --- |
| Windows 10 / 11 | [Windows 使用指南](guides/windows.md) | 安装、设置、输入模式、词库导入导出、语音和排障 |
| macOS 12+ | [macOS 使用指南](guides/macos.md) · [语音输入](guides/macos-voice.md) | 安装、原生设置、快捷键、数据与卸载 |
| Linux / IBus | [Linux 使用指南](guides/linux.md) | 安装启用、设置、桌面工具、语音命令与排障 |
| iOS | [Apple 平台仓库](https://github.com/metasequoiaime/MSIME-Apple) | 开发中的键盘扩展，以实现仓进展为准 |

Windows 指南是官网对应正文的唯一来源。修改用户说明请在本仓提交，MSIME-Web 通过固定的 Docs 子模块渲染；构建/API 说明仍由各代码仓库维护。指南按已提交源码核对，具体安装版本可能有所不同，请结合 Release 阅读。

仓库边界与数据来源见[公共仓库与平台架构](architecture/repositories.md)。

## 开发与维护

| 说明 | 阅读入口 |
| --- | --- |
| 仓库职责、输入链路、数据来源与问题归属 | [公共仓库与平台架构](architecture/repositories.md) |
| 平台消费版本与迁移验收 | [平台接入矩阵](architecture/platform-adoption.md) |
| CI 检查、健康审计与排障 | [持续集成与自动维护](development/continuous-integration.md) |
| 固定提交的跨平台兼容预检 | [Engine 接入预检](development/platform-preflight.md) |
| 过往实施、核对依据与界面示例 | [历史记录](archive/README.md) |

## 开源代码

代码与模块入口集中在[架构说明](architecture/repositories.md)，本页不维护第二份仓库清单。

## 手动构建

本仓只有文档。模块 API、依赖安装、构建与测试命令以各实现仓库的 README 和工作流为准；跨仓预检见[开发与维护](#开发与维护)。

## 截图

[历史界面示例](archive/interface-examples.md)已归档；当前使用方式请看对应平台指南。

## 贡献

文档修改遵循[本仓贡献指南](CONTRIBUTING.md)。通用贡献流程、安全策略、治理与招募统一在[组织仓库](https://github.com/metasequoiaime/.github)维护。

## 感谢

历史签名证书支持：[Certum](https://www.certum.eu/en/)。

## 许可协议

GPL-3.0.
