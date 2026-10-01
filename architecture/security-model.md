# 安全模型与信任边界

输入法看得见用户敲下的每一个键，包括密码。这决定了它的信任门槛远高于普通桌面应用，也决定了「哪些东西是不可信输入、它们在哪一层被检查」值得单独写清楚，而不是散落在代码注释里。

这份文档描述的是**当前**的边界与加固措施，以及已知的薄弱处。按 2026-10-01 的已发布版本与源码整理：Windows 为 [msime-windows](https://github.com/metasequoiaime/msime-windows) 的 v0.9.3，macOS 为 [msime](https://github.com/metasequoiaime/msime) 的 v0.50.0-build.9，Linux 仍是已归档的 msime-linux 发布的 v0.8.2。

## 有哪些进程，各自拿到什么

| 组件 | 运行位置 | 拿到的东西 |
| --- | --- | --- |
| TSF 文本服务 DLL（Windows） | **加载进宿主进程内**——Chrome、QQ、Office 等 | 按键、组字状态，以及宿主进程的整个地址空间 |
| Server（Windows） | 独立进程 | 组字串、候选、词库、配置、全部网络调用 |
| 设置程序与 WebView2（Windows） | 设置页是独立进程 `MetasequoiaImeSettings.exe`，用 WebView2 渲染；候选窗、悬浮工具栏、托盘菜单默认由 Server 用 Direct2D 绘制，“界面渲染”改选 WebView2 后才由 Server 托管的 WebView2 渲染 | 设置页读写的配置；选用 WebView2 时还有候选与工具栏状态 |
| IBus engine（Linux） | 独立进程，由 ibus-daemon 拉起 | 同 Server |
| InputMethodKit 输入源（macOS） | 独立进程 | 同 Server |

**TSF DLL 是最关键的一层**：它不是一个和宿主对话的独立进程，而是运行在宿主进程内。这里一次越界写坏的是 Chrome 或 Office 自己的内存。这也是为什么它用静态 CRT 构建——避免与宿主的 MSVCP140 / VCRUNTIME140 冲突（QQ 上常见的崩溃特征）。

msime 主仓的 `develop` 上另有 iOS、Android、HarmonyOS 宿主、Linux 的 Fcitx5 插件和一套开发中的 Windows 宿主，目前都没有正式版（iOS 只有 TestFlight 预发布构建），不在上表中。其中 Fcitx5 插件的边界与 IBus 不同：它是被 `fcitx5` 进程加载的 addon，引擎和词库运行在 `fcitx5` 进程内，引擎一崩，所有应用同时失去输入法。

## 不可信输入从哪里进来

按「用户没有写过、但会被本项目解析」排列：

| 输入 | 来源 | 在哪一层检查 |
| --- | --- | --- |
| 皮肤包（`skin.toml` 与 CSS） | 用户从任意来源下载的文件夹 | 注入前对 CSS 里的 URL 做分类：包内相对路径内联、`data:` 与文档内片段保留、任何指向包外的一律丢弃；远程 `@import` 前置剥离；候选窗文档带 CSP |
| 词库文件（`.db`、批量导入的文本） | 发布资产、用户自行导入 | 发布资产按 `product-lock.json` 里提交的 SHA256 校验，来源 commit 与格式清单一并核对；导入文本按列数、音节数、权重校验并报错到行 |
| IPC 负载 | TSF DLL 与 Server 之间 | 契约定义在 msime-windows 仓内引擎的 `engine/contracts/`，帧有明确的 magic、版本与长度上限 |
| 配置文件（`config.toml` / `config.ini`） | 用户可编辑 | 与出厂模板合并，缺失项取默认值 |
| 云端点返回的内容 | 第三方服务 | 异步处理；失败、超时、焦点变化与过期结果一律丢弃，不阻塞也不改变本地输入 |
| Windows 的更新清单（`update.json`） | msime.app | 校验版本号形状、release URL 前缀、安装包名与 SHA256 的格式 |

## 会出网的地方

完整清单、各自的默认状态与关闭方法见[隐私说明](https://msime.app/privacy/)以及各平台仓库的 `PRIVACY.md`。安全视角下要点有三：

- **不需要凭据、装完就生效的联网功能各平台不同。** Windows 与 Linux 是云候选，发往 Google 的 input-tools 服务；Windows 首次安装时会在安装器里就此询问。macOS 发布版没有云候选，但输入法首次激活时会向 `api.msime.app` 自动注册一个匿名账号，候选翻译默认开启，用这个账号把当前页还没有释义的候选词发到 `api.msime.app` 取释义；另有 Sparkle 的更新检查，不携带输入内容。`api.msime.app` 的服务端是 [msime-cloud](https://github.com/metasequoiaime/msime-cloud)，它声明输入正文、音频和凭据不写日志或磁盘。
- Windows 上 AI 联想、候选翻译、语音输入的配置开关出厂为开，但随包的 token 是占位符且运行时拒绝占位符，**在用户填入真实 token 之前不会发出任何请求**。
- Linux 运行时只接受 HTTPS 端点。Windows 不是这样：自定义翻译端点同时接受 `http://` 与 `https://`。

## 产物的信任链

| 环节 | 现状 |
| --- | --- |
| 构建来源证明 | msime-windows 的发布工作流会为安装包生成 GitHub build provenance attestation，可用 `gh attestation verify` 核对；但当前的 v0.9.x 安装包查不到 attestation，macOS 与 Linux 的发布包也没有 |
| 校验值 | 每个发布资产都有 SHA256；官网下载页直接展示 Windows 安装包的那一份 |
| 代码签名 | Windows 安装包带 Authenticode 签名（证书由 Certum 签发）；macOS 发布包用 Developer ID 签名并经 Apple 公证；**Linux 包未签名**，无 detached 签名 |
| macOS 更新通道 | Sparkle appcast 经 Ed25519 签名，解包前校验——在 app 本体的 Developer ID 签名之外，更新路径另有一层独立的签名校验 |
| 供应链 | 所有工作流的第三方 action 按 40 位 SHA 固定；submodule 与词库均按 commit / tag + 摘要锁定 |

**Linux 包未签名是当前最薄的一环。** Windows 与 macOS 的发布包已经签名，Linux 包仍然只能靠 SHA256 核对完整性。

## 本机数据

设置、学习词频、自造词、日志都只写在当前用户目录下，仅受该账号的文件权限保护。

各平台的 API token 存放方式不同，这是一处已知的不对等：macOS 的语音服务 token 存在钥匙串，Linux 用桌面 Secret Service，**Windows 明文存放在 `config.toml`**，任何能读到该文件的程序都能读到 token。macOS 发布版的候选翻译若改用腾讯云，SecretId 与 SecretKey 同样是明文，存在输入法的 NSUserDefaults（`~/Library/Preferences/app.msime.inputmethod.MetasequoiaIME.plist`）里，不进钥匙串。macOS 自动注册的匿名账号也不在钥匙串里：它的凭据和换来的会话令牌存在 `~/Library/Application Support/metasequoiaime/` 下的 `anonymous-account.json` 与 `anonymous-session.json`（0600），不进钥匙串，以该用户身份运行的程序都能读到并冒用这个匿名账号。msime 主仓 `develop` 上的 Linux 宿主（尚未发布）不再用 Secret Service，服务商凭据和账号会话存在 0600 权限的文件里，不加密静态数据。

日志记录的是进程生命周期与错误状况，不记录输入内容。Windows 与 Linux 的剪贴板历史默认关闭，Linux 上即使开启也会跳过密码管理器标记为机密的条目；macOS 发布版没有本地剪贴板历史，只有需手动开启的云剪贴板，且只上传用户明确添加的条目。

## 已知薄弱处

写在这里是因为它们真实存在，不是因为已经解决：

- Linux 包未签名（见上）。
- Windows 的 API token 明文存储；macOS 发布版的腾讯云翻译凭据同样明文存在偏好设置里。
- TSF 前端的自有测试覆盖很薄，且没有 sanitizer 覆盖：msime-windows 的 ASan/UBSan job 只覆盖 Server 中与宿主无关的解析与策略代码。macOS 宿主在 msime 的 CI 里有 ASan/UBSan job；已发布的 Linux 包来自 msime-linux，它的 CI 有 ASan/UBSan job，迁入 msime 后的 Linux 宿主 CI 还没有。
- 未做可复现构建，用户无法独立验证二进制与公开源码的对应关系。构建来源证明可以部分缓解，但当前的发布包都没有附带（见上）。

## 报告问题

疑似漏洞请按[组织安全策略](https://github.com/metasequoiaime/.github/blob/main/SECURITY.md)私下上报；msime 主仓有自己的 [SECURITY.md](https://github.com/metasequoiaime/msime/blob/develop/SECURITY.md)，涉及该仓库时以它为准。不要提交公开 issue，也不要在公开渠道附带敏感的验证材料。
