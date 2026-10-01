# 平台固定版本

合仓前的平台接入矩阵（C++ Engine 经 gitlink 接入 Windows、Apple、Linux 三个仓库的固定提交、接入 PR 与迁移验收计划）已于 2026-10-01 移到[归档](../archive/2026-09-06-platform-adoption-matrix.md)，早期预检失败记录见[历史预检记录](../archive/platform-adoption-preflight-2026-09-06.md)。本页只说明各产品现在如何固定引擎与词库。仓库职责见[公共仓库与平台架构](repositories.md)；模块 API 和构建命令仍由实现仓维护。

## 核对范围

2026-10-01 核对以下提交与 Release。“开发分支”描述源码，不代表已经进入安装包；已发布行为以 Release 的来源提交为准。

| 仓库 | 开发分支提交 | 已发布版本 |
| --- | --- | --- |
| msime | [`develop` 36403ece](https://github.com/metasequoiaime/msime/tree/36403ece77baf535b51110ed417c7f1fee1851b6) | macOS `v0.50.0-build.9`（Latest，2026-09-18）；其后的预发布自动构建 macOS `v0.50.0-build.11` 与 iOS `ios-v0.50.0-build.14`（TestFlight） |
| msime-windows | [`develop` ad29d7eb](https://github.com/metasequoiaime/msime-windows/tree/ad29d7eb041c02877493c3130232cf82e138b8e3) | `v0.9.3`（Latest，2026-09-30） |

## msime

开发分支上，输入引擎是仓内的 Rust crate `crates/engine`，随 workspace 构建和测试，没有引擎锁文件，也不再从外部拉取源码归档（见 msime `ARCHITECTURE.md` 的“上游 Engine”）。随包词库由 `resources/desktop-dictionary.lock.json` 固定：其中八个产物来自 msime-engine 的 `dict-v2.0.1` Release，`sentence-model.safetensors` 来自 chinese-ime-lm 的 `model-v1`，每个产物记录 URL、长度和 SHA-256，`source_commit` 记录这批词库的来源提交。建库输入另由 `resources/dictionary-sources.lock.json` 固定，辅助码表在 `resources/helpcodes/`。

已发布的版本早于这套结构：上表三个 Release 的来源提交仍以 `engine-lock.json` 固定 C++ MSIME-Engine `6f235a51`，以 `product-lock.json` 固定词库 `dict-v1.0.0`。开发分支的 Rust 引擎和 `dict-v2.0.1` 词库尚未随 Release 发布。

## msime-windows

`v0.9.3` 与开发分支相同：Engine 源码已并入仓内 `engine/`，不再有 gitlink，来源提交（MSIME-Engine `c810d201`，导入时版本 0.9.0）和裁剪清单记在 `engine/UPSTREAM.md`。此后 `engine/` 按本仓的一方代码评审修改，不再跟随上游更新；辅助码在 `engine/helpcode/`，同样由本仓提交固定。

`product-lock.json` 只记录仍来自仓外的输入，即词库 Release：msime-engine 的 `dict-v1.0.0`、`source_commit` `d0dc0c2b594b5540b5de99ad12085c786410626e` 和每个产物的 SHA256。构建时下载词库并按清单校验；刷新、校验与发布前检查见 msime-windows 的 `docs/product-release.md`。

两个产品的引擎各自演进，固定版本不要求一致。词库则要求一致：组织健康审计（`.github` 的 `tools/product_locks.py`，见[持续集成与自动维护](../development/continuous-integration.md)）读取两仓默认分支，比较 msime-windows 的 `product-lock.json` 与 msime 的 `resources/desktop-dictionary.lock.json`，词库来源仓库、tag、`source_commit` 或共有产物的摘要不一致即报错。2026-10-01 核对时 msime 开发分支固定 `dict-v2.0.1`，msime-windows 固定 `dict-v1.0.0`，2026-09-30 的组织审计运行因此失败。产物摘要以各自锁文件为准，本页不维护第二份摘要清单。

## 更新本页

1. 获取两个产品仓的最新默认分支和 Release，记录被核对的提交与版本，别只读取当前工作目录。
2. 从开发分支提交和 Release 来源提交分别读取上面列出的锁文件和来源说明；数据 tag 相同不能证明代码相同。
3. 分别写明“开发分支”和“已发布”状态，任何一项缺证据就保留待核对说明，不把开发分支的结构写成已发布。
