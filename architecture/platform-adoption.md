# 平台接入矩阵与迁移验收

本页跟踪公共能力从 Engine 到平台产品的接入过程。仓库职责见[公共仓库与平台架构](repositories.md)；模块 API 和构建命令仍由实现仓维护。

## 核对范围

2026-09-06 从远端获取 `main` 后核对以下提交。表格描述源码接入，不声称这些提交已经进入安装包；已发布行为还需核对平台 Release 的来源提交和产物清单。本地工作分支、未提交子模块和未跟踪目录不作为主分支证据。

| 仓库 | 核对的主分支提交 |
| --- | --- |
| 组织规范 | [d4bcc027](https://github.com/metasequoiaime/.github/tree/d4bcc027f753cf7054205f6505bab70131d3de85) |
| Engine | [7d5169d0](https://github.com/metasequoiaime/MSIME-Engine/tree/7d5169d0083cd87e6588b6399602755d0aefde2d) |
| Windows | [2562026a](https://github.com/metasequoiaime/MSIME-Windows/tree/2562026a2a1c393e387fcd8ccd2dc30024a44463) |
| Apple | [72e6c652](https://github.com/metasequoiaime/MSIME-Apple/tree/72e6c652cf1f768c14f61210c2a2b50763271caf) |
| Linux | [c51aa9c5](https://github.com/metasequoiaime/MSIME-Linux/tree/c51aa9c5071484b1a5b3b83836b54bbfa5cec7ef) |

## 已合入的接入状态

| 能力 | Windows | macOS | iOS | Linux |
| --- | --- | --- | --- | --- |
| 公共输入会话 | Server 的 `EngineInputSession` 包装 `InputSession` | 原生控制器持有 `InputSession` | `shared/apple-bridge` 包装 `InputSession` | `InputController` 持有 `InputSession` |
| Engine 固定提交 | `cc21e429` | `cc21e429` | 使用 Apple 固定版本 | `cc21e429` |
| 词库发布来源 | Engine `dict-v1.0.0` | Engine `dict-v1.0.0` | 从 Apple 锁定数据生成 mobile profile | Engine `dict-v1.0.0` |
| 锁定的词库文件 | `msime.db`、`english.db`、`others.db`、日语模型及授权文件 | `msime.db` | compact 拼音词库 | `msime.db`、`english.db`、`others.db` |
| 辅助码来源 | 固定 Engine 的 `helpcode/` | 固定 Engine 的 `helpcode/` | 不因 Engine 包含辅助码就自动开放 UI 功能 | 固定 Engine 的 `helpcode/` |
| 公共语音 | 公共采集/协议，Server 保留传输与上屏 | 公共录音/识别及可选 Whisper，原生权限与上屏 | 无语音入口 | 公共协议，Linux 保留采集命令与 HTTP 传输 |
| 新 `Session` facade / 显式 `RuntimePaths` | 尚未接入 | 尚未接入 | 尚未接入 | 尚未接入 |
| 完整 desktop 资源 ZIP | 尚未接入 | 尚未接入 | 不直接采用 desktop profile | 尚未接入 |

三个产品的词库来源提交均为 `d0dc0c2b594b5540b5de99ad12085c786410626e`，Engine gitlink 为 `cc21e42916cf93ce43051fec474e87eeeba305bf`。两者用途不同，不要求相等。产物摘要以各自 `product-lock.json` 为准，本页不维护第二份摘要清单。

核对入口：各平台 `product-lock.json`、`vendor/MetasequoiaImeEngine` gitlink、根 `CMakeLists.txt`（Windows 为 `server/CMakeLists.txt`），以及 Windows `server/src/session/`、Apple `shared/apple-bridge/` 与 `platforms/macos/src/MetasequoiaInputController.mm`、Linux `src/InputController.*`。iOS 移动产品见 `platforms/ios/scripts/prepare_dictionary.py`。

## 在途生产者与兼容风险

[Engine #34](https://github.com/metasequoiaime/MSIME-Engine/pull/34) 在核对时仍开放，提交 `e31d726ed9828f5e1e05fa60c3b36c9ee3769aa2` 提供 `Session`、`RuntimePaths`、会话隔离和完整资源包。它属于待合入能力，不能在组织规范中写成已被三端采用。

该提交的[Linux ASan/UBSan 检查](https://github.com/metasequoiaime/MSIME-Engine/actions/runs/34004189681/job/101408326006)失败于 `data_path` 的词典缓冲区泄漏；[Google-PinyinIME-Rev #4](https://github.com/metasequoiaime/Google-PinyinIME-Rev/pull/4) 是在途修复。完成条件是上游修复合入、Engine 更新到已合入提交、最终 Engine 提交的检查通过并合入。旧提交的绿勾和本地脏子模块不构成完成证据。

直接更新 Engine 还存在这些接入差距：

| 差距 | 当前消费点 | 迁移要求 |
| --- | --- | --- |
| 全局辅助码变成每会话配置 | Windows `EngineInputSession::ApplyConfiguration`、macOS `reloadSessionFromPreferences`、Linux `select_active_helpcode_schema` | 对活动会话使用实例配置，验证两个不同方案的会话互不影响；不能只改 gitlink |
| 新 facade 只暴露动作和快照 | Windows 适配器还暴露原始序列 setter；macOS/Linux 依赖更宽的配置和查询接口 | 逐项映射现有行为，先确认公共接口能表达现有功能，禁止为迁移删除设置或复制状态机 |
| 资源与用户数据生命周期分开 | 平台安装器、数据准备和偏好重置流程 | 明确不可变资源、持久用户数据、可丢弃缓存；准备/切换时暂停会话与写入，失败继续使用旧数据 |
| desktop 准备入口不是通用移动安装器 | macOS 当前仅带主库，iOS 是 compact 拼音产品 | 保持平台已有资源规格；显式配置路径，不为了复用桌面准备函数强制增加英文/日语资源 |
| 异步请求新增会话身份 | Windows/Linux 在线宿主与回调 | 保留完整请求身份，验证相同拼音的新会话也不能接收旧响应 |

## 接入预检实测

2026-09-06 在 macOS / AppleClang 21 上，把上述 Apple 和 Linux 主分支的既有可移植测试与两个 Engine 提交分别编译、运行。源码和第三方子模块均从提交对象导出，不使用工作区的未提交修改。

| Engine 提交 | iOS bridge 回归 | Linux 控制器回归 |
| --- | --- | --- |
| 平台当前固定 `cc21e429` | 通过 | 通过 |
| 待接入 `e31d726e` | 通过 | 失败：`The independent Quanpin helpcode schema was not selected before input.` |

失败由原有控制器测试报告，验证了辅助码实例配置需要随 Engine 更新同步迁移。此结果覆盖可移植 C++ 行为，不是 Linux IBus 原生实测，也不证明 iOS 模拟器、设备或安装已经通过。

使用组织仓库的[接入预检脚本](https://github.com/metasequoiaime/.github/blob/main/scripts/check-platform-adoption.py)重复比较，命令与范围见其 [README](https://github.com/metasequoiaime/.github#engine-接入预检)。脚本保留每次运行的 `evidence.json`、构建与测试日志，失败返回非零；平台原生 CI 仍是后续验收的一部分。

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
