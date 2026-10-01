# 公共仓库与平台架构

公共部分已合并：多平台主仓 [msime](https://github.com/metasequoiaime/msime)（原 MSIME-Apple 改名）承载 Android、iOS、macOS、Linux、HarmonyOS 的原生宿主和开发中的 Windows 宿主，以及共享的 Rust 输入引擎、宿主接口和 React 设置界面；原 C++ 版 MSIME-Engine 已移植为 msime 的 `crates/engine`，Linux 前端从 msime-linux 并入 msime，这两个旧仓库均已归档。目前 msime 只发布过 macOS 版本和 iOS 预发布版；官网的 Linux 安装包仍来自已归档 msime-linux 的 v0.8.2 Release，Android 与 HarmonyOS 尚无 Release。已发布的 Windows 产品仍来自 [msime-windows](https://github.com/metasequoiaime/msime-windows)，Windows 组件和从 MSIME-Engine 导入的引擎都在它的目录中。词库源数据（原 MSIME-Dict，以及 msime-customdict 的内容）统一到 [msime-dictionary](https://github.com/metasequoiaime/msime-dictionary)。云端后端、官网、用户文档、扩展包、皮肤、语言模型、pinyin_cpp、pinyin_python 等仓库保持独立。

各产品当前如何固定引擎与词库，以及开发分支与已发布版本的差别，见[平台固定版本](platform-adoption.md)。合仓前的接入矩阵与迁移验收计划已移到[归档](../archive/2026-09-06-platform-adoption-matrix.md)。

| 位置 | 当前职责 |
| --- | --- |
| msime 的 `platforms/<os>/` | 各系统的原生宿主，也是最终安装、被系统识别为输入法的产品本体：原生权限、界面、焦点、按键与文本提交 |
| msime 的 `crates/host-api` | 版本化 C 接口与线程绑定的会话句柄，各平台宿主都链接它 |
| msime 的 `crates/input-runtime` | 会话编排、焦点取消、候选分页和带代次的选择；不复制组词逻辑 |
| msime 的 `crates/engine` | 纯 Rust 输入引擎：组合状态、各输入方案、词库查询与学习回放 |
| msime 的 `crates/client-core` | 与按键链路平行的宿主无关业务：本地配置、固定资源分代安装、账号、云与 AI、皮肤、词库、语音、翻译等 |
| msime 的 `packages/ui`、`apps/desktop` | 共享 React 设置页与 Tauri 承载层，由原生宿主按需承载，不单独作为任何平台的产品 |
| msime 的 `shared/` | macOS 与 iOS 共用的桥接、公共语音服务适配（`shared/voice/`）和 Windows 与各宿主共用的 IPC 契约头文件（`shared/contracts/`） |
| msime 的 `crates/dict-builder`、`resources/` | 词库构建器 `msime-dict-build`；词库、模型的锁文件，辅助码表（`resources/helpcodes/`）和手工维护的表情、符号、快捷短语源数据 |
| [msime-dictionary](https://github.com/metasequoiaime/msime-dictionary) | 词库源数据：基础词库（`cn/`、`en/`）、人工维护词条与翻译（`custom/`）、专业词库（`packs/`）；以 `sources-v*` release 发布，由 msime 的 `crates/dict-builder` 按锁定版本构建 |
| msime-windows 的 windows/、server/、engine/、ui/、ui-html/、installer/ | 已发布 Windows 产品的全部一方源码，统一仓库与 CI；DLL/Server 仍隔进程通信，GUI 保持通用库边界；`engine/` 是从 MSIME-Engine 导入后按 Windows 专门化的引擎，含辅助码、跨进程契约与语音模块 |
| msime-windows 的 log/、skins/、experiments/tsf-edit-control/ | 日志库、外部皮肤合集与 TSF 编辑控件实验 |
| [msime-cloud](https://github.com/metasequoiaime/msime-cloud) | Go 后端：云候选、AI 联想、翻译与语音识别接口 |
| [msime-plugins](https://github.com/metasequoiaime/msime-plugins) | 社区扩展包：按键音效、旋律、背景音乐、`/` 指令表与打字特效 |
| msime-docs | 用户指南、产品架构、跨仓开发维护说明与历史归档 |
| msime-web | 官网呈现、Docs 固定版本渲染与下载元数据 |
| .github | 组织政策、公共配置、执行规则和跨仓自动化 |

独立实验与资源入口：[pinyin_cpp](https://github.com/metasequoiaime/pinyin_cpp)、[pinyin_python](https://github.com/metasequoiaime/pinyin_python)、[chinese-ime-lm](https://github.com/metasequoiaime/chinese-ime-lm)（语言模型与评测集，msime 锁定的整句神经模型从它的 release 下载）、[Google-PinyinIME-Rev](https://github.com/metasequoiaime/Google-PinyinIME-Rev)、[皮肤示例](https://github.com/metasequoiaime/msime-skin-example)、[PC 端皮肤](https://github.com/metasequoiaime/msime-skins) 和 [Homebrew tap](https://github.com/metasequoiaime/homebrew-tap)（目前还没有 cask：msime 开发分支的 macOS 发布流程会在公证的 DMG 正式版发布后写入 `Casks/msime.rb`，本文核对时尚未发生）。

msime 的引擎随 workspace 一起构建和测试，没有子模块或锁文件；它以 C++ 引擎录下的行为基准对照。msime-windows 的 `engine/` 不再跟踪上游，按本仓一等代码维护。两份引擎因此各自演进，修改前先确认问题出在哪个产品。语音方面，msime 的 `shared/voice/` 是服务商识别与润色的公共适配，录音、豆包会话与波形浮层归各平台宿主；msime-windows 的录音、WAV 编码与识别/润色协议在 `engine/voice/`。服务商配置和原生交互由平台维护。

词库发布与代码版本分开：`dict-*` release 提供数据和 `dictionary-manifest.json`，平台锁文件记录发布地址、源提交及文件摘要。msime 开发分支的 `resources/desktop-dictionary.lock.json` 固定已归档 msime-engine 的 `dict-v2.0.1`，尚未随 Release 发布；已发布的 macOS 版本、iOS 预发布版、Linux（msime-linux v0.8.2）和 msime-windows 都按各自的 `product-lock.json` 使用 msime-engine 的 `dict-v1.0.0`。之后的词库由 msime 的 `release-dictionary.yml` 用 `msime-dict-build` 构建，以 `dict-vX.Y.Z` 发布在 msime，本文核对时尚未发布过这类版本。源数据进入词库的顺序是：msime-dictionary 发布 `sources-vX.Y.Z`，msime 在 `resources/dictionary-sources.lock.json` 升级引用并构建 `dict-v*`，各平台再升级各自锁定的词库版本。锁文件中的源提交是生成数据的提交，可能与平台代码版本不同，不能互相冒充。更新数据时先验证摘要、格式及来源，再按平台既有流程回放用户词库。

已归档的旧仓库保留历史和已有 Release，不再接受修改：msime-engine、msime-linux、msime-helpcode、MetasequoiaVoiceInput、MSIME-Windows-Server、MSIME-UI、MSIME-UiHtml、MSIME-Installer、MetasequoiaImeLog 和 TsfEditControl。当前修改应提交到 msime、msime-windows 对应目录或 msime-dictionary。Windows 组件（Server、UI、UiHtml、Installer、Log、TsfEditControl）合入 msime-windows 时保留了原始提交历史；`engine/` 是 MSIME-Engine `c810d201` 的快照导入，没有带历史，来源与裁剪清单记在其 `engine/UPSTREAM.md`。msime 的 Rust 引擎是移植而非导入，参考实现的来源记在其 `tools/engine-golden/README.md`。msime-customdict 尚未归档，但内容已并入 msime-dictionary，msime 的词库构建不再读取它。msime-dictionary（原 MSIME-Dict）的旧版 `dict-2026.09.05` 不会被新产物覆盖。2026-09-30 词库源数据从 msime-engine 的 `dictionary/` 与 msime-customdict 移入 msime-dictionary 时直接复制内容，来源提交记录在该仓 README 与提交说明中。

## 一次输入如何流转

1. 平台前端接收按键，判断当前焦点、输入模式及是否交给宿主应用。
2. 引擎管理输入会话与预编辑，按方案查询词库，生成候选并处理选择与学习。msime 中的调用链是宿主 → `crates/host-api` → `crates/input-runtime` → `crates/engine`；msime-windows 由 Server 调用仓内的 `engine/`。
3. 平台前端显示候选，并通过各自的输入框架提交文本：Windows 使用 TSF，macOS 使用 InputMethodKit，Linux 使用 IBus（msime 中另有并列的 Fcitx5 入口），iOS 使用键盘扩展，Android 使用输入法服务，HarmonyOS 使用 InputMethodExtensionAbility。
4. 在线候选或语音异步返回时，平台与会话层需要确认请求仍属于当前输入会话，避免把旧结果写入新的输入位置。

组织执行约束见 [AGENTS.md](https://github.com/metasequoiaime/.github/blob/main/AGENTS.md)，CI 操作见[持续维护说明](../development/continuous-integration.md)。

Windows 的 TSF DLL 加载到宿主应用中，Server 在独立进程中调度引擎和窗口，两者通过 `engine/contracts/` 定义的版本化管道协议通信。`ui/` 提供通用 GUI 能力，不依赖输入法业务、Server 全局状态或词库；原生窗口由 Server 拥有，`ui-html/` 提供页面；业务动作和配置持久化由 Server 处理。合仓不改变这些运行时边界。DLL 的静态 CRT 与 Server 的动态 CRT 构建树保持独立。msime 中开发中的 `platforms/windows` 保留同样的 DLL/Server 进程与协议边界，契约头文件在 `shared/contracts/`。

平台共享引擎不代表用户功能完全相同：是否携带额外词库、是否链接语音模块、如何录音与上屏、设置哪些入口，都由平台产品决定。例如 macOS 语音可选本地 Whisper，已发布的 Linux 版本（msime-linux v0.8.2）通过独立命令输出转写文本，Windows 使用原生菜单与录音快捷键。

## 词库与用户数据

| 数据 | 权威来源 | 消费方式 |
| --- | --- | --- |
| 输入引擎与公共契约 | msime 的 `crates/engine` 与 `shared/contracts/`；msime-windows 的 `engine/`（含 `engine/contracts/`） | 随所在仓库源码一起构建；已发布的 macOS 版本与 iOS 预发布版仍按当时的 `engine-lock.json` 固定 C++ MSIME-Engine 提交，Linux v0.8.2 通过 msime-linux 的 Engine 子模块固定 |
| 基础词库 | 锁定的词库 Release 和来源提交 | 校验文件摘要、格式及来源后安装 |
| 辅助码 | msime 的 `resources/helpcodes/`；msime-windows 的 `engine/helpcode/` | 随平台产品安装，不在词库发布里 |
| 移动端词库 | 与桌面相同的 `resources/desktop-dictionary.lock.json` | iOS、Android、HarmonyOS 打包时按同一锁文件校验名称、长度与 SHA-256 |
| 用户词条与学习 | 本机运行时数据 | 由平台持久化，在基础数据更新时按平台流程回放 |

新版数据清单和平台锁文件共同描述发布输入。历史无清单数据只能使用消费者明确支持的兼容入口，不能把任意缺少清单的数据视为合法旧版。具体格式字段、校验命令和兼容测试留在 msime 的 [`crates/dict-builder`](https://github.com/metasequoiaime/msime/tree/develop/crates/dict-builder)、msime-windows 的 [`engine/contracts`](https://github.com/metasequoiaime/msime-windows/tree/develop/engine/contracts) 与各平台构建文档。

## 文档与网站发布

`guides/` 下的 Windows、macOS、macOS 语音、Linux 指南和常见问题（连同 `guides/zh-TW/` 的繁体版本）是官网对应页面正文的唯一维护源。Web 的 `vendor/MSIME-Docs` 固定某个 Docs 提交，`src/page-docs.tsx` 与 `src/page-faq.tsx` 导入正文并生成目录；所以合入 Docs 后，官网不会自动读取浮动的 `main`，还需要更新 Web 的 gitlink。

文档发布顺序是：先在 Docs 完成内容核对并合并，再由 Web 更新到已合并提交，检查渲染后发布。其余指南目前可从本仓 README 阅读；网站是否增加这些页面由 Web 的路由和导航决定。

网站下载和应用更新元数据来自已发布 Release，属于 Web；本仓链接到下载入口，不维护另一份安装包清单。模块构建命令、API 字段与开发测试步骤留在实现仓，避免版本更新后产生多份互相冲突的操作说明。

## 问题应提交到哪里

| 现象 | 提交位置 |
| --- | --- |
| 某个宿主中的按键、光标、候选窗、焦点、安装或上屏问题 | Windows 提交到 [msime-windows](https://github.com/metasequoiaime/msime-windows)；macOS、iOS、Linux、Android、HarmonyOS 提交到 [msime](https://github.com/metasequoiaime/msime) |
| 候选顺序、组词、联想或纠错问题 | Windows 版对应 [msime-windows/engine](https://github.com/metasequoiaime/msime-windows/tree/develop/engine)，其他平台对应 [msime/crates/engine](https://github.com/metasequoiaime/msime/tree/develop/crates/engine) |
| 词条、拼音或权重问题 | [msime-dictionary](https://github.com/metasequoiaime/msime-dictionary) |
| 词库构建问题 | [msime](https://github.com/metasequoiaime/msime) 的 `crates/dict-builder` |
| 辅助码筛选或数据问题 | Windows 版对应 [msime-windows/engine/helpcode](https://github.com/metasequoiaime/msime-windows/tree/develop/engine/helpcode)，其他平台对应 [msime](https://github.com/metasequoiaime/msime) 的 `resources/helpcodes/` 与 `crates/engine` |
| Windows 设置页面、托盘、工具栏 | [msime-windows/server](https://github.com/metasequoiaime/msime-windows/tree/develop/server) |
| Windows 安装、升级与卸载 | [msime-windows/installer](https://github.com/metasequoiaime/msime-windows/tree/develop/installer) |
| 云候选、AI、翻译、语音识别的服务端接口 | [msime-cloud](https://github.com/metasequoiaime/msime-cloud) |
| 扩展包内容 | [msime-plugins](https://github.com/metasequoiaime/msime-plugins) |
| 用户与开发说明缺失或过时 | [Docs](https://github.com/metasequoiaime/msime-docs) |
| 官网导航、下载链接或渲染 | [Web](https://github.com/metasequoiaime/msime-web) |

不能判断时先提交到遇到问题的平台仓库，维护者再转移。反馈记录平台与产品版本、宿主应用和最小复现步骤；开发者核对该版本使用的引擎来源和词库锁定版本，不用主分支结果推断旧安装包行为。提交流程与隐私要求遵循[组织贡献指南](https://github.com/metasequoiaime/.github/blob/main/CONTRIBUTING.md)。
