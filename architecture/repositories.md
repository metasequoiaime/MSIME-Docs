# 公共仓库与平台架构

公共部分已合并：Engine、Dict、CustomDict、HelpCode 和 VoiceInput 统一到 MSIME-Engine。Windows 组件统一到 MSIME-Windows 的目录中；Windows、Linux、Apple 保留各自的平台边界；官网、用户文档、皮肤示例、pinyin_cpp、pinyin_python 和 n-gram 等仓库保持独立。

| 位置 | 当前职责 |
| --- | --- |
| MSIME-Engine 的 core、schemes 等目录 | 输入会话、候选、查询和学习 |
| Engine/dictionary | 桌面和移动词库构建，根 build_profile.py 是公开入口 |
| Engine/dictionary/custom | 可验证的自定义词典与翻译包 |
| Engine/helpcode | 辅助码数据与来源说明 |
| Engine/voice | 公共识别、文本整理、音频采集与可选 Whisper 接口 |
| Engine/contracts | 输入法协议、WebView 消息和词库格式契约 |
| MSIME-Apple、MSIME-Linux | 原生权限、界面、焦点、按键与文本提交 |
| MSIME-Windows 的 windows/、server/、ui/、ui-html/、installer/ | Windows 产品源码，统一仓库与 CI；DLL/Server 仍隔进程通信，GUI 保持通用库边界 |
| MSIME-Windows 的 log/、experiments/tsf-edit-control/ | 日志库与 TSF 编辑控件实验 |
| MSIME-Docs、MSIME-Web | 用户文档正文、官网呈现与下载导航 |

平台通过固定 Engine 提交消费代码和辅助码。语音模块按需构建，普通输入引擎不会因合仓而强制加载录音设备、网络服务或 Whisper 模型。macOS 的菜单与快捷键调用公共库，Windows Server 的录音采集也使用同一实现；服务商配置和原生交互由平台维护。

词库发布与代码版本分开：Engine 的 `dict-*` release 提供数据和 `dictionary-manifest.json`，各平台的 `product-lock.json` 记录发布仓库、源提交及文件摘要。平台的 Engine gitlink 可能与生成数据的提交不同，不能互相冒充。更新数据时先验证摘要、格式及来源，再按平台既有流程回放用户词库。

合仓使用保留历史的导入方式，原始提交和第三方声明仍可追溯。10 个被替代的旧公共/Windows 组件仓库均已归档，保留历史和已有 Release；当前修改应提交到 Engine 或 Windows 对应目录。旧版 `MSIME-Dict/dict-2026.09.05` 不会被新产物覆盖。

## 一次输入如何流转

1. 平台前端接收按键，判断当前焦点、输入模式及是否交给宿主应用。
2. Engine 管理输入会话与预编辑，按方案查询词库，生成候选并处理选择与学习。
3. 平台前端显示候选，并通过各自的输入框架提交文本：Windows 使用 TSF，macOS 使用 InputMethodKit，Linux 使用 IBus；iOS 使用键盘扩展适配层。
4. 在线候选或语音异步返回时，平台与会话层需要确认请求仍属于当前输入会话，避免把旧结果写入新的输入位置。

Windows 的 TSF DLL 加载到宿主应用中，Server 在独立进程中调度引擎和窗口，两者通过 Engine 定义的版本化管道协议通信。`ui/` 提供通用 GUI 能力，`ui-html/` 提供页面；业务动作和配置持久化由 Server 处理。合仓不改变这些运行时边界。

平台共享引擎不代表用户功能完全相同：是否携带额外词库、是否链接语音模块、如何录音与上屏、设置哪些入口，都由平台产品决定。例如 macOS 语音可选本地 Whisper，Linux 当前通过独立命令输出转写文本，Windows 使用原生菜单与录音快捷键。

## 词库与用户数据

| 数据 | 权威来源 | 消费方式 |
| --- | --- | --- |
| 输入引擎与公共契约 | Engine 固定提交 | 平台子模块／产品锁定版本 |
| 基础词库 | 锁定的词库 Release 和来源提交 | 校验文件摘要、格式及来源后安装 |
| 辅助码 | 固定 Engine 中的 `helpcode/` | 随平台产品安装，与引擎版本核对 |
| 移动词库 | 经校验的源数据库与固定构建器 | 使用 `build_profile.py` 的移动 profile 生成 |
| 用户词条与学习 | 本机运行时数据 | 由平台持久化，在基础数据更新时按平台流程回放 |

新版数据清单和平台锁文件共同描述发布输入。历史无清单数据只能使用消费者明确支持的兼容入口，不能把任意缺少清单的数据视为合法旧版。具体格式字段、校验命令和兼容测试留在 [Engine 契约目录](https://github.com/metasequoiaime/MSIME-Engine/tree/main/contracts) 与各平台构建文档。

## 文档与网站发布

`MSIME-Docs/guides/windows.md` 是 Windows 指南正文的唯一维护源。Web 的 `vendor/MSIME-Docs` 固定某个 Docs 提交，`src/docs.ts` 导入正文并生成目录；所以合入 Docs 后，官网不会自动读取浮动的 `main`，还需要更新 Web 的 gitlink。

文档发布顺序是：先在 Docs 完成内容核对并合并，再由 Web 更新到已合并提交，检查渲染后发布。macOS/Linux 指南目前可从本仓 README 阅读；网站是否增加这些页面由 Web 的路由和导航决定。

网站下载和应用更新元数据来自已发布 Release，属于 Web；本仓链接到下载入口，不维护另一份安装包清单。模块构建命令、API 字段与开发测试步骤留在实现仓，避免版本更新后产生多份互相冲突的操作说明。

## 问题应提交到哪里

- 安装、切换输入源、焦点、按键与上屏问题：对应平台仓库。
- 查询、候选顺序、词条与辅助码问题：Engine；不能判断时先提交到遇到问题的平台仓库。
- 用户说明缺失或过时：Docs。
- 官网布局、导航、下载链接或渲染问题：Web。

用户反馈时记录平台与产品版本、宿主应用和最小复现步骤；开发者再核对相应产品锁定的 Engine 和数据版本。不要仅用 Engine 当前主分支的结果推断旧安装包行为。
