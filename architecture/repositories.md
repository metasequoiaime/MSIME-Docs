# 公共仓库与平台架构

本次先合并公共部分：Engine、Dict、CustomDict、HelpCode 和 VoiceInput 统一到 MSIME-Engine。Windows、Linux、Apple 继续保留各自的平台边界；官网、用户文档、皮肤示例、pinyin_cpp、pinyin_python 和 n-gram 等仓库保持独立。

| 位置 | 当前职责 |
| --- | --- |
| MSIME-Engine 的 core、schemes 等目录 | 输入会话、候选、查询和学习 |
| Engine/dictionary | 桌面和移动词库构建，根 build_profile.py 是公开入口 |
| Engine/dictionary/custom | 可验证的自定义词典与翻译包 |
| Engine/helpcode | 辅助码数据与来源说明 |
| Engine/voice | 公共识别、文本整理、音频采集与可选 Whisper 接口 |
| Engine/contracts | 输入法协议、WebView 消息和词库格式契约 |
| MSIME-Apple、MSIME-Linux | 原生权限、界面、焦点、按键与文本提交 |
| MSIME-Windows、Server、UiHtml、UI、Installer | 现有 Windows 产品组件，本阶段保持进程和仓库边界 |
| MSIME-Docs、MSIME-Web | 用户文档正文、官网呈现与下载导航 |

平台通过固定 Engine 提交消费代码和辅助码。语音模块按需构建，普通输入引擎不会因合仓而强制加载录音设备、网络服务或 Whisper 模型。macOS 的菜单与快捷键调用公共库，Windows Server 的录音采集也使用同一实现；服务商配置和原生交互由平台维护。

词库发布与代码版本分开：Engine 的 `dict-*` release 提供数据和 `dictionary-manifest.json`，各平台的 `product-lock.json` 记录发布仓库、源提交及文件摘要。平台的 Engine gitlink 可能与生成数据的提交不同，不能互相冒充。更新数据时先验证摘要、格式及来源，再按平台既有流程回放用户词库。

合仓使用保留历史的导入方式，原始提交和第三方声明仍可追溯。旧仓库保留历史和已有 Release；当前修改应提交到 Engine 对应目录。旧版 `MSIME-Dict/dict-2026.09.05` 不会被新产物覆盖。
