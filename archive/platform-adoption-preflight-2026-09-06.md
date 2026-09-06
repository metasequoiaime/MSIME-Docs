# 2026-09-06 早期平台接入预检记录

本页保留早期迁移中的失败与修复过程，不描述当前接入状态。当前证据见[平台接入矩阵](../architecture/platform-adoption.md)。原始记录未完整列出每次消费者提交，不能作为最新消费者组合的验收证据。

历史提交 `e31d726e` 的[Linux ASan/UBSan 检查](https://github.com/metasequoiaime/MSIME-Engine/actions/runs/34004189681/job/101408326006)失败于 `data_path` 的词典缓冲区泄漏。[Google-PinyinIME-Rev #4](https://github.com/metasequoiaime/Google-PinyinIME-Rev/pull/4) 随后已合入，Engine `2c8fdea6` 更新了固定版本，其[Linux 内存检查通过](https://github.com/metasequoiaime/MSIME-Engine/actions/runs/34005092776/job/101410700125)，修复随 #34 合入。


2026-09-06 在 macOS / AppleClang 21 上，把当时 Apple 和 Linux 主分支的既有可移植测试与下列 Engine 提交分别编译、运行。源码和第三方子模块均从提交对象导出，不使用工作区的未提交修改。

| Engine 提交 | iOS bridge 回归 | Linux 控制器回归 |
| --- | --- | --- |
| 当时平台固定 `cc21e429` | 通过 | 通过 |
| 待接入 `e31d726e` | 通过 | 失败：`The independent Quanpin helpcode schema was not selected before input.` |
| 泄漏修复后 `2c8fdea6` | 通过 | 同一辅助码断言失败 |

失败由原有控制器测试报告，验证了辅助码实例配置需要随 Engine 更新同步迁移。此结果覆盖可移植 C++ 行为，不是 Linux IBus 原生实测，也不证明 iOS 模拟器、设备或安装已经通过。

