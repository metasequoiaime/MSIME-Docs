# 平台接入矩阵与迁移验收

本页跟踪公共能力从 Engine 到平台产品的接入过程。仓库职责见[公共仓库与平台架构](repositories.md)；模块 API 和构建命令仍由实现仓维护。

## 核对范围

2026-09-06 从远端获取 `main` 后核对以下提交。表格描述源码接入，不声称这些提交已经进入安装包；已发布行为还需核对平台 Release 的来源提交和产物清单。本地工作分支、未提交子模块和未跟踪目录不作为主分支证据。

| 仓库 | 核对的主分支提交 |
| --- | --- |
| 组织规范 | [0f1ececc](https://github.com/metasequoiaime/.github/tree/0f1ececc474c5eb30277861b6f45ab41929f0c0d) |
| Engine | [3a23c6e5](https://github.com/metasequoiaime/MSIME-Engine/tree/3a23c6e55c46d5898554b88fd856c570ebf0870a) |
| Windows | [2d51c538](https://github.com/metasequoiaime/MSIME-Windows/tree/2d51c5383925ce8174a60f5527cbda5ff03b7d9f) |
| Apple | [8926a474](https://github.com/metasequoiaime/MSIME-Apple/tree/8926a474b41551c61de3a24d4dda9f52c1ec24a6) |
| Linux | [13a07a55](https://github.com/metasequoiaime/MSIME-Linux/tree/13a07a55c6f7d402d43b3c024b8a24b0d0cb306d) |

## 已合入的接入状态

| 能力 | Windows | macOS | iOS | Linux |
| --- | --- | --- | --- | --- |
| 公共输入会话 | Server 的 `EngineInputSession` 包装 `InputSession` | 原生控制器持有公共 `Session` 和值快照 | `shared/apple-bridge` 使用公共 `Session` | `InputController` 持有公共 `Session` 和值快照 |
| Engine 固定提交 | `c63ba774` | `020e906a` | 使用 Apple 固定版本 | `020e906a` |
| 词库发布来源 | Engine `dict-v1.0.0` | Engine `dict-v1.0.0` | 从 Apple 锁定数据生成 mobile profile | Engine `dict-v1.0.0` |
| 锁定的词库文件 | `msime.db`、`english.db`、`others.db`、日语模型及授权文件 | `msime.db` | compact 拼音词库 | `msime.db`、`english.db`、`others.db` |
| 辅助码来源 | 固定 Engine 的 `helpcode/` | 固定 Engine 的 `helpcode/` | 不因 Engine 包含辅助码就自动开放 UI 功能 | 固定 Engine 的 `helpcode/` |
| 公共语音 | 公共采集/协议，Server 保留传输与上屏 | 公共录音/识别及可选 Whisper，原生权限与上屏 | 无语音入口 | 公共协议，Linux 保留采集命令与 HTTP 传输 |
| 运行时路径 | 兼容 InputSession，捕获旧目录布局 | 公共 Session，捕获旧目录布局 | 公共 Session，捕获旧目录布局 | 公共 Session，捕获旧目录布局；完整路径贯通见 #92 |
| 完整 desktop 资源 ZIP | 尚未接入 | 尚未接入 | 不直接采用 desktop profile | 尚未接入 |

三个产品的词库来源提交均为 `d0dc0c2b594b5540b5de99ad12085c786410626e`。Engine gitlink 见上表，与词库来源用途不同，不要求相等。产物摘要以各自 `product-lock.json` 为准，本页不维护第二份摘要清单。

核对入口：各平台 `product-lock.json`、`vendor/MetasequoiaImeEngine` gitlink、根 `CMakeLists.txt`（Windows 为 `server/CMakeLists.txt`），以及 Windows `server/src/session/`、Apple `shared/apple-bridge/` 与 `platforms/macos/src/MetasequoiaInputController.mm`、Linux `src/InputController.*`。iOS 移动产品见 `platforms/ios/scripts/prepare_dictionary.py`。

## 已合入生产者与兼容风险

[Engine #34](https://github.com/metasequoiaime/MSIME-Engine/pull/34) 已合入主分支 `c63ba774ac03c9ff8ba82776d50e97d27fd459a3`，提供 `Session`、`RuntimePaths`、会话隔离和完整资源包构建能力。Windows #178、Apple #272 已合入并接入此版本。公共桌面接口扩展 #37 已合入 `020e906a`，Apple #274 和 Linux #84 已固定此版本并迁移桌面控制器。

[Engine #38](https://github.com/metasequoiaime/MSIME-Engine/pull/38) 已合入 `59b76c05`，在准备数据代际前拒绝重叠的资源、用户和缓存目录。[Engine #39](https://github.com/metasequoiaime/MSIME-Engine/pull/39) 已合入 `3a23c6e5`，完整资源包的真实消费者验证已进入建库 CI，覆盖学习回放、回退、缓存清理、失败准备和资源摘要不变。该验证复用同一份产物测试代际流程，不代表平台跨版本安装已经完成。

早期解码器泄漏及辅助码预检失败的发现与修复见[历史预检记录](../archive/platform-adoption-preflight-2026-09-06.md)。
直接更新 Engine 还存在这些接入差距：

| 差距 | 当前消费点 | 迁移要求 |
| --- | --- | --- |
| 辅助码实例配置已接入 | Windows #178、Apple #272/#274、Linux #84 | 后续升级继续保留不同会话、不同方案的隔离回归 |
| 新 facade 只暴露动作和快照 | Windows 适配器还暴露原始序列 setter，事件处理器自行管理部分候选和造词流程；macOS/Linux 已迁移 | 逐项映射现有行为，先确认公共接口能表达现有功能，禁止为迁移删除设置或复制状态机 |
| 资源与用户数据生命周期分开 | 平台安装器、数据准备和偏好重置流程 | 明确不可变资源、持久用户数据、可丢弃缓存；准备/切换时暂停会话与写入，失败继续使用旧数据 |
| desktop 准备入口不是通用移动安装器 | macOS 当前仅带主库，iOS 是 compact 拼音产品 | 保持平台已有资源规格；显式配置路径，不为了复用桌面准备函数强制增加英文/日语资源 |
| 异步请求新增会话身份 | Linux 已通过 Session 回传完整请求；Windows 仍需迁移在线候选接入 | 保留完整请求身份，验证相同拼音的新会话也不能接收旧响应 |

## 接入预检实测

2026-09-06 在 macOS / AppleClang 21 上，使用组织预检脚本从提交对象导出 Engine、两个第三方子模块及消费者源码，编译并运行真实消费者的可移植测试。

| Engine 提交 | Apple 消费者提交 / iOS bridge | Linux 消费者提交 / 控制器 |
| --- | --- | --- |
| #41 的 `0655f93f663b8dfc536a78efacee159d147fe928`（待合入） | `8926a474b41551c61de3a24d4dda9f52c1ec24a6`：通过 | `13a07a55c6f7d402d43b3c024b8a24b0d0cb306d`：通过 |

这次预检没有修改消费者 gitlink 或产品锁。它证明上述两个可移植消费者能与待评审 Engine 组合，不证明 Windows、原生输入法宿主或安装升级已经通过。早期失败记录保留在[归档](../archive/platform-adoption-preflight-2026-09-06.md)。

使用组织仓库的[接入预检脚本](https://github.com/metasequoiaime/.github/blob/main/scripts/check-platform-adoption.py)重复比较，命令与范围见[接入预检说明](../development/platform-preflight.md)。脚本保留每次运行的 `evidence.json`、构建与测试日志，失败返回非零；平台原生 CI 仍是后续验收的一部分。

## 消费者 PR 与实际验证

Engine 由 fanlusky、houko 共同维护。公共接口扩展 [Engine #37](https://github.com/metasequoiaime/MSIME-Engine/pull/37) 已合入 `020e906abe5a6af78cec70a1dced8789812eac52`，三平台、Linux ASan/UBSan、资源构建验证及公共语音 CI 均通过。消费者以下述已合入提交固定依赖，不再依赖本地实验提交。

| 平台 / 评审 | 状态与改动 | 验证范围 |
| --- | --- | --- |
| [Windows #178](https://github.com/metasequoiaime/MSIME-Windows/pull/178) | 已合入；固定 c63ba774，隔离活动会话辅助码筛选与候选提示 | [CI](https://github.com/metasequoiaime/MSIME-Windows/actions/runs/34006530164) 全部通过，覆盖 Server 真实词库回归、TSF Win32/x64 构建及管道探针 |
| [Apple #272](https://github.com/metasequoiaime/MSIME-Apple/pull/272) | 已合入；iOS bridge 使用 Session，macOS 仍用 InputSession 并隔离码表 | [CI](https://github.com/metasequoiaime/MSIME-Apple/actions/runs/34006179512) 全部通过；本地 38 项 CTest、iOS 模拟器构建和引导页 XCTest 通过 |
| [Linux #84](https://github.com/metasequoiaime/MSIME-Linux/pull/84) | 已合入 `fd7f313`；固定 020e906a，控制器、设置验证和在线请求头使用公共 Session | 最终 PR 提交 `57eed98` 的[四组 Ubuntu CI](https://github.com/metasequoiaime/MSIME-Linux/actions/runs/34008164931) 均通过 |
| [Apple #274](https://github.com/metasequoiaime/MSIME-Apple/pull/274) | 已合入 `d28d490`；固定 020e906a，macOS 控制器、候选导航与五笔自动提交使用公共 Session | PR 提交 `6804122` 的[macOS 双架构与 iOS CI](https://github.com/metasequoiaime/MSIME-Apple/actions/runs/34007476842) 通过；本地通用构建、38 项 CTest、签名和架构校验通过 |
| [Linux #92](https://github.com/metasequoiaime/MSIME-Linux/pull/92) | 待合入 `b93d765`；基于最新 main，显式路径贯通控制器、词库检查与翻译侧文件 | 本地 25 项可移植 CTest 通过；Linux 专属 `/proc/self/exe` 测试在 macOS 失败，干净 main 同样失败。新提交的 Ubuntu CI 待确认，旧提交结果不替代它 |

公共候选管理仍在生产者侧评审：[Engine #40](https://github.com/metasequoiaime/MSIME-Engine/pull/40) 增加置顶动作，[Engine #41](https://github.com/metasequoiaime/MSIME-Engine/pull/41) 增加删除动作并把工作词库删除与日志写入放进同一事务。两者尚未进入本页核对的 main，也尚未被 Windows 消费；不计作平台迁移完成。

这些改动保持既有数据 Release 与摘要，尚未完成完整资源包和用户数据代际生命周期接入。Apple 引导页 XCTest 不证明实际宿主中的第三方键盘交互，Windows PR CI 不替代安装后 uiAccess 与原生焦点验证。是否随产品发布还需按实际 Release 来源另行核对。

## 推进顺序和完成证据

以下是待办与验收条件，不是已完成报告。每项完成时补实际 PR、最终提交和验证链接。

| 顺序 | 工作 | 完成证据 |
| --- | --- | --- |
| 1 | 收尾上游解码器与 Engine #34 | 已合入提交可从默认分支到达；该提交的根 CTest、三平台构建、Linux 内存检查、资源和真实词库消费检查通过 |
| 2 | 先核对平台辅助码与异步兼容，再更新 Engine 固定版本 | 对活动会话修改配置、双会话隔离、同输入旧响应拒绝回归；Windows 同步产品锁和 Web 契约生成副本 |
| 3 | 从较小的 iOS bridge 开始使用 `Session` | 字符、退格、候选、原文提交、取消、模式切换和未消费后缀回归；模拟器宿主/扩展构建；真实设备启动与内存测量单独记录 |
| 4 | 按平台迁移 macOS、Linux、Windows 适配 | 原有能力逐项保留；公共状态只由 Engine 推进；原生焦点、吃键、文本提交和设置变更验证 |
| 5 | 分别接入资源生命周期 | 已发布固定摘要的资源；学习不修改基础库；坏摘要拒绝；升级失败保留旧目录；回退重放新学习；清缓存不清用户词库 |
| 6 | 完成交付与文档同步 | 平台最终提交的组合验证和产物清单；按实际 Release 更新已发布状态；用户说明进入 Docs，Web 只更新已合入 Docs gitlink |

接口迁移和资源打包可以分 PR 评审，但最终都要验证平台安装与运行时组合。资料和计划完成不等于平台迁移完成。Windows 的 DLL/Server 进程隔离、GUI 通用边界以及各平台原生权限与上屏职责继续保持。

## 更新这份矩阵

1. 获取生产者与消费者最新默认分支，先记录被审查的 commit，别只读取当前工作目录。
2. 从消费者提交读取 Engine gitlink、产品锁和实际调用代码；检查生产者提交可达性。数据 tag 相同不能证明代码接口相同。
3. 分别填写“生产者已合入”“平台已接入”“最终组合已验证”“已发布”；任何一项缺证据就保留待验证状态。
4. 链接同一最终提交的测试结果。只构建引擎、只检查摘要或只通过静态检查，都不能证明宿主焦点、安装回放或输入行为。
5. 更新涉及的平台行与源码依据；保留历史 PR 中的验证记录，不覆盖旧版来源。组织规范只链接本页，不再复制一份状态表。
