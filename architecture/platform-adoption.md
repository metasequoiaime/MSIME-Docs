# 平台接入矩阵与迁移验收

本页跟踪公共能力从 Engine 到平台产品的接入过程。仓库职责见[公共仓库与平台架构](repositories.md)；模块 API 和构建命令仍由实现仓维护。

## 核对范围

2026-09-06 从远端获取 `main` 后核对以下提交。表格描述源码接入，不声称这些提交已经进入安装包；已发布行为还需核对平台 Release 的来源提交和产物清单。本地工作分支、未提交子模块和未跟踪目录不作为主分支证据。

| 仓库 | 核对的主分支提交 |
| --- | --- |
| 组织规范 | [b9a23bb9](https://github.com/metasequoiaime/.github/tree/b9a23bb91e71e816e07a87c60812fdc55f388153) |
| Engine | [c63ba774](https://github.com/metasequoiaime/MSIME-Engine/tree/c63ba774ac03c9ff8ba82776d50e97d27fd459a3) |
| Windows | [41ef263f](https://github.com/metasequoiaime/MSIME-Windows/tree/41ef263f3c2c1ef107f6df6be616df66c3ceba7e) |
| Apple | [b1c6e651](https://github.com/metasequoiaime/MSIME-Apple/tree/b1c6e651929c0b8d99f03b9fb498ce37033e4854) |
| Linux | [c746385a](https://github.com/metasequoiaime/MSIME-Linux/tree/c746385a35a892a544684e6db53ea48155ba898d) |

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

## 已合入生产者与兼容风险

[Engine #34](https://github.com/metasequoiaime/MSIME-Engine/pull/34) 已合入主分支 `c63ba774ac03c9ff8ba82776d50e97d27fd459a3`，提供 `Session`、`RuntimePaths`、会话隔离和完整资源包构建能力。三端主分支仍固定旧引擎，因此生产者已合入不等于产品已接入。

历史提交 `e31d726e` 的[Linux ASan/UBSan 检查](https://github.com/metasequoiaime/MSIME-Engine/actions/runs/34004189681/job/101408326006)失败于 `data_path` 的词典缓冲区泄漏。[Google-PinyinIME-Rev #4](https://github.com/metasequoiaime/Google-PinyinIME-Rev/pull/4) 随后已合入，Engine `2c8fdea6` 更新了固定版本，其[Linux 内存检查通过](https://github.com/metasequoiaime/MSIME-Engine/actions/runs/34005092776/job/101410700125)，修复随 #34 合入。以下保留预检历史，用于说明消费者必须同步修改的原因。

直接更新 Engine 还存在这些接入差距：

| 差距 | 当前消费点 | 迁移要求 |
| --- | --- | --- |
| 全局辅助码变成每会话配置 | Windows `EngineInputSession::ApplyConfiguration`、macOS `reloadSessionFromPreferences`、Linux `select_active_helpcode_schema` | 对活动会话使用实例配置，验证两个不同方案的会话互不影响；不能只改 gitlink |
| 新 facade 只暴露动作和快照 | Windows 适配器还暴露原始序列 setter；macOS/Linux 依赖更宽的配置和查询接口 | 逐项映射现有行为，先确认公共接口能表达现有功能，禁止为迁移删除设置或复制状态机 |
| 资源与用户数据生命周期分开 | 平台安装器、数据准备和偏好重置流程 | 明确不可变资源、持久用户数据、可丢弃缓存；准备/切换时暂停会话与写入，失败继续使用旧数据 |
| desktop 准备入口不是通用移动安装器 | macOS 当前仅带主库，iOS 是 compact 拼音产品 | 保持平台已有资源规格；显式配置路径，不为了复用桌面准备函数强制增加英文/日语资源 |
| 异步请求新增会话身份 | Windows/Linux 在线宿主与回调 | 保留完整请求身份，验证相同拼音的新会话也不能接收旧响应 |

## 接入预检实测

2026-09-06 在 macOS / AppleClang 21 上，把上述 Apple 和 Linux 主分支的既有可移植测试与下列 Engine 提交分别编译、运行。源码和第三方子模块均从提交对象导出，不使用工作区的未提交修改。

| Engine 提交 | iOS bridge 回归 | Linux 控制器回归 |
| --- | --- | --- |
| 平台当前固定 `cc21e429` | 通过 | 通过 |
| 待接入 `e31d726e` | 通过 | 失败：`The independent Quanpin helpcode schema was not selected before input.` |
| 泄漏修复后 `2c8fdea6` | 通过 | 同一辅助码断言失败 |

失败由原有控制器测试报告，验证了辅助码实例配置需要随 Engine 更新同步迁移。此结果覆盖可移植 C++ 行为，不是 Linux IBus 原生实测，也不证明 iOS 模拟器、设备或安装已经通过。

使用组织仓库的[接入预检脚本](https://github.com/metasequoiaime/.github/blob/main/scripts/check-platform-adoption.py)重复比较，命令与范围见其 [README](https://github.com/metasequoiaime/.github#engine-接入预检)。脚本保留每次运行的 `evidence.json`、构建与测试日志，失败返回非零；平台原生 CI 仍是后续验收的一部分。

## 平台迁移分支（尚未合入或发布）

2026-09-06 以下草稿 PR 均固定已合入的 Engine `c63ba774`，保持原有 `dict-v1.0.0` 来源与摘要。其测试证据不替代上面的主分支状态。

| 平台 / 评审 | 实际改动 | 验证与尚未覆盖范围 |
| --- | --- | --- |
| [Linux #84](https://github.com/metasequoiaime/MSIME-Linux/pull/84)，`1d36ea0` | 对活动会话配置辅助码；取消每次按键重载；仍使用兼容 InputSession | 本地可移植 CTest 25 项通过；[Ubuntu 24.04/26.04、amd64/arm64 CI](https://github.com/metasequoiaime/MSIME-Linux/actions/runs/34006048460) 四组均通过；未声称桌面宿主人工交互已验证 |
| [Apple #272](https://github.com/metasequoiaime/MSIME-Apple/pull/272)，`dee7659` | iOS bridge 使用 Session 动作与快照；macOS 仍用 InputSession，筛选和提示使用控制器所属码表 | 本地通用 macOS 构建、签名检查和 38 项 CTest 通过；iOS 模拟器构建及引导页 XCTest 通过；[原生 CI](https://github.com/metasequoiaime/MSIME-Apple/actions/runs/34006179512) arm64、x86_64、iOS 全部通过；真实设备内存和键盘宿主交互待验证 |
| [Windows #178](https://github.com/metasequoiaime/MSIME-Windows/pull/178)，`0c63c06` | 对活动会话配置辅助码，候选提示读取该适配器持有码表；缓存未变化方案；同步产品锁 | 契约同步、产品锁校验和 18 项 Python 回归通过；[Windows 原生 CI](https://github.com/metasequoiaime/MSIME-Windows/actions/runs/34006530164) 全部通过，覆盖 Server 真实词库回归、TSF Win32/x64 构建和两种架构的管道探针 |

后续 [Engine #37](https://github.com/metasequoiaime/MSIME-Engine/pull/37)（`424b745`）补充 `finish(first_index)`、辅助码开关/方案验证，以及独立英文模式快照。生产者本地 13 项根 CTest 通过，原生 CI 仍待结果。基于 Linux #84 的独立本地检出已将控制器、设置验证和在线请求头迁到公共 Session；25 项可移植 CTest 及新增方案开关往返回归通过。该候选尚未发布到消费者分支，须等 #37 合入后固定最终主分支提交，并重新验证原生 IBus/GTK；本地临时测试使用未合入生产者不能作为平台已接入证据。

三个 PR 均未完成新的资源包和用户数据代际生命周期接入。Apple 的引导页 XCTest 不能证明第三方键盘在实际宿主中的交互；Windows 的 PR CI 也不代表签名、安装与 uiAccess 验证。后续应在同一最终提交补齐证据。

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
