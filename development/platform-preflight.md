# Engine 接入预检

这份预检用于合仓前 C++ Engine 升级时比较 MSIME-Apple 与 MSIME-Linux 的可移植测试。合仓后 MSIME-Engine 与 MSIME-Linux 已归档，msime 主仓的引擎改为仓内 Rust crate `crates/engine`，预检所需的测试源码已不在当前仓库中，原说明已于 2026-10-01 移到[归档](../archive/2026-09-06-platform-preflight.md)。

msime 现行的验证分层与本地门禁见 msime 的 [ARCHITECTURE.md](https://github.com/metasequoiaime/msime/blob/develop/ARCHITECTURE.md#验证)；各产品如何固定引擎与词库见[平台固定版本](../architecture/platform-adoption.md)。
