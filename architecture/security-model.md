# 安全模型与信任边界

输入法看得见用户敲下的每一个键，包括密码。这决定了它的信任门槛远高于普通桌面应用，也决定了「哪些东西是不可信输入、它们在哪一层被检查」值得单独写清楚，而不是散落在代码注释里。

这份文档描述的是**当前**的边界与加固措施，以及已知的薄弱处。按 2026-09-06 的源码整理。

## 有哪些进程，各自拿到什么

| 组件 | 运行位置 | 拿到的东西 |
| --- | --- | --- |
| TSF 文本服务 DLL（Windows） | **加载进宿主进程内**——Chrome、QQ、Office 等 | 按键、组字状态，以及宿主进程的整个地址空间 |
| Server（Windows） | 独立进程 | 组字串、候选、词库、配置、全部网络调用 |
| WebView2 宿主（Windows） | 独立进程 | 候选窗、悬浮工具栏、托盘菜单、设置页的渲染 |
| IBus engine（Linux） | 独立进程，由 ibus-daemon 拉起 | 同 Server |
| InputMethodKit 输入源（macOS） | 独立进程 | 同 Server |

**TSF DLL 是最关键的一层**：它不是一个和宿主对话的独立进程，而是运行在宿主进程内。这里一次越界写坏的是 Chrome 或 Office 自己的内存。这也是为什么它用静态 CRT 构建——避免与宿主的 MSVCP140 / VCRUNTIME140 冲突（QQ 上常见的崩溃特征）。

## 不可信输入从哪里进来

按「用户没有写过、但会被本项目解析」排列：

| 输入 | 来源 | 在哪一层检查 |
| --- | --- | --- |
| 皮肤包（`skin.toml` 与 CSS） | 用户从任意来源下载的文件夹 | 注入前对 CSS 里的 URL 做分类：包内相对路径内联、`data:` 与文档内片段保留、任何指向包外的一律丢弃；远程 `@import` 前置剥离；候选窗文档带 CSP |
| 词库文件（`.db`、批量导入的文本） | 发布资产、用户自行导入 | 发布资产按 `product-lock.json` 里提交的 SHA256 校验，来源 commit 与格式清单一并核对；导入文本按列数、音节数、权重校验并报错到行 |
| IPC 负载 | TSF DLL 与 Server 之间 | 契约定义在引擎的 `contracts/`，帧有明确的 magic、版本与长度上限 |
| 配置文件（`config.toml` / `config.ini`） | 用户可编辑 | 与出厂模板合并，缺失项取默认值 |
| 云端点返回的内容 | 第三方服务 | 异步处理；失败、超时、焦点变化与过期结果一律丢弃，不阻塞也不改变本地输入 |
| 更新清单（`update.json`） | msime.app | 校验版本号形状、release URL 前缀、安装包名与 SHA256 的格式 |

## 会出网的地方

完整清单、各自的默认状态与关闭方法见[隐私说明](https://msime.app/privacy/)以及各平台仓库的 `PRIVACY.md`。安全视角下要点有三：

- **云候选是唯一不需要凭据、装完就生效的联网功能**，发往 Google 的 input-tools 服务。Windows 首次安装时会在安装器里就此询问。
- AI 联想、候选翻译、语音输入的配置开关出厂为开，但随包的 token 是占位符且运行时拒绝占位符，**在用户填入真实 token 之前不会发出任何请求**。
- 运行时只接受 HTTPS 端点。

## 产物的信任链

| 环节 | 现状 |
| --- | --- |
| 构建来源证明 | Windows 安装包带 GitHub build provenance attestation，可用 `gh attestation verify` 核对 |
| 校验值 | 每个发布资产都有 SHA256；官网下载页直接展示 Windows 安装包的那一份 |
| 代码签名 | **三个平台的产物目前均未签名。** Windows 会被 SmartScreen 拦截且 uiAccess 失效，macOS 未经公证会被 Gatekeeper 拦截，Linux 包无 detached 签名 |
| macOS 更新通道 | Sparkle appcast 经 Ed25519 签名，解包前校验——即使 app 本体未签名，更新路径本身是可验证的 |
| 供应链 | 所有工作流的第三方 action 按 40 位 SHA 固定；submodule 与词库均按 commit / tag + 摘要锁定 |

**未签名是当前最薄的一环。** 签名流水线的代码已经就绪，缺的是证书与仓库 secret。

## 本机数据

设置、学习词频、自造词、日志都只写在当前用户目录下，仅受该账号的文件权限保护。

各平台的 API token 存放方式不同，这是一处已知的不对等：macOS 用钥匙串，Linux 用桌面 Secret Service，**Windows 明文存放在 `config.toml`**。任何能读到该文件的程序都能读到 token。

日志记录的是进程生命周期与错误状况，不记录输入内容。剪贴板历史三平台均默认关闭；Linux 上即使开启也会跳过密码管理器标记为机密的条目。

## 已知薄弱处

写在这里是因为它们真实存在，不是因为已经解决：

- 三个平台的产物均未签名（见上）。
- Windows 的 API token 明文存储。
- TSF 前端的自有测试覆盖很薄，且此前无 sanitizer 覆盖；Linux 前端有 ASan/UBSan job，Apple 与 Windows 的对应覆盖正在补齐。
- 未做可复现构建，用户无法独立验证二进制与公开源码的对应关系（构建来源证明部分缓解了这一点）。

## 报告问题

疑似漏洞请按[组织安全策略](https://github.com/metasequoiaime/.github/blob/main/SECURITY.md)私下上报，不要提交公开 issue，也不要在公开渠道附带敏感的验证材料。
