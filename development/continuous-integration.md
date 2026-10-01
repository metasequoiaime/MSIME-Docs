# 持续集成与自动维护

本文说明组织级 CI 的使用与排障；[工作流](https://github.com/metasequoiaime/.github/tree/main/.github/workflows)、[审计脚本](https://github.com/metasequoiaime/.github/blob/main/tools/audit_repositories.py)和[健康策略](https://github.com/metasequoiaime/.github/blob/main/tools/health-policy.json)仍在 `.github` 仓库维护。以下组织审计命令均从 `.github` 仓库根目录执行。

本规范覆盖组织当前维护的（未归档的公开）仓库。msime-engine、msime-linux 和拆分前的 Windows 组件仓等已归档仓库保留历史工作流，不再纳入审计。Engine 合仓后由两个产品仓各自验证：msime 的共享 Rust 引擎在 `crates/engine`，GitHub Actions 只在各平台宿主构建中编译它，`cargo test` 与 clippy 由本地 `scripts/verify-local.sh` 运行（见[本地复现](#本地复现)）；msime-windows 发布版使用的引擎内嵌在 `engine/`，由其 CI 的 `Engine` 任务用产品实际数据测试。

## 所有维护仓库的共同检查

- `Repository quality` 在 PR、`develop` 与 `main` 的 push、合并队列和手动运行时使用 actionlint 检查工作流及嵌入 Shell。ShellCheck 的 warning/error 阻断合并，风格建议不作为门禁。
- PR 的 Dependency review 拒绝新增 high/critical 已知依赖漏洞；它依赖 GitHub dependency graph，不能替代原生库构建测试。
- `CodeQL` 扫描实际语言及 Actions。msime、msime-windows、msime-web、msime-docs、msime-dictionary 和 .github 每天定时扫描一次（UTC 15:00），不随 PR 或 push 运行；Google-PinyinIME-Rev、pinyin_cpp、pinyin_python 和 msime-skin-example 的 `codeql.yml` 设计为在 PR 和每周定时扫描，但自 2026-09-06 起其 `on:` 块残留一行缩进错误的 `branches`，工作流文件无法解析，目前不会运行（msime-skin-example 的修复在 PR #6 中待合并）；chinese-ime-lm 另在 `main` 的 push 上扫描。C/C++ 使用源码分析；msime 的 Swift 通过未签名 iOS 模拟器编译提取。C/C++ 源码分析不能替代各平台编译或发现所有宏配置下的问题。
- Actions 使用完整提交 SHA；Dependabot 通过 PR 更新。依赖更新须通过相同的构建测试，不能仅凭机器人身份绕过门禁。
- CI 默认只读、使用 GitHub 托管 runner，设定超时并取消过时的同事件运行。发布流程按其实际用途单独保留写权限；msime-windows 的发布构建、签名和打包任务运行在自托管 Windows runner 上。

msime-cloud、msime-customdict 和 msime-plugins 目前没有 `quality.yml`、`codeql.yml` 和 `.github/dependabot.yml`，msime-skins 与 homebrew-tap 没有 `.github` 目录；下面的组织审计会把这些缺口如实报出，不要把它们当作已满足上述检查。

`Organization CI coverage` 每日检查组织当前所有未归档公开仓库的默认分支，发现缺失的功能 CI、质量检查、CodeQL、Dependabot 配置或被停用的工作流。它还比较 msime-windows 的 `product-lock.json` 与 msime 的 `resources/desktop-dictionary.lock.json` 固定的词库 Release，tag、source commit 或资产摘要不一致即报错。可以手动运行，或用 `python tools/audit_repositories.py` 在已登录 gh CLI 的环境复现。PR 中运行只读覆盖检查及健康审计回归；已审查的 main 上另用组织现有 `RELEASE_PLEASE_TOKEN` 执行只读健康检查（所有 API 请求都是 GET），避免把管理权限交给 PR 代码。

健康审计按 `tools/health-policy.json` 检查各仓实际必需任务、检查结果的 GitHub Actions 来源、禁止强推/删除、PR 规则、安全扫描、依赖图、漏洞提醒和私密漏洞报告。它也检测最近三次已完成运行连续失败、排队/运行超过三小时、工作流被停用或创建一天后仍没有运行。取消的过期运行不计为失败。定时 CodeQL 十天内须成功，Web 下载清单同步（`sync-downloads.yml`）六小时内须成功，组织审计两天内须成功；只随 push 运行的仓库在没有改动时不会因闲置误报。

失败会使 Actions 任务变红，完整发现列表作为 `repository-health` JSON artifact 保留 30 天，由 GitHub 现有通知设置通知维护者。审计不自动关闭安全功能、改规则或重跑发布。新增活跃公开仓库会要求维护者补充健康策略，已归档和私有仓库不纳入。

## 各仓库的功能验证

| 仓库 | 自动验证 |
| --- | --- |
| msime | `ci.yml`：actionlint、Dependency review 和读取仓内文件做断言的契约脚本（`scripts/run-checks.sh`）；`ci-macos.yml`：macOS 15 arm64/x86_64 构建原生宿主并跑 ctest、arm64 上的 ASan/UBSan，以及 ad-hoc 签名后核对 bundle 架构和 Info.plist 输入法键；`ci-ios.yml`：iOS 模拟器构建与单测和 PR 范围的界面用例，进 `main` 的 PR 另跑完整界面套件（含设置与皮肤分片）和 Intel runner 上的手写识别；`ci-platforms.yml`：按改动范围检查 Android 宿主契约与 JVM 冒烟、Linux IBus/Fcitx5 宿主在固定容器内的构建与 ctest、HarmonyOS 类型检查、键盘逻辑测试和设置页构建、Windows MinGW x64 交叉构建（宿主、TSF DLL 与原生测试）及 Windows 发布与安装脚本测试；只改 README、PRIVACY、CHANGELOG 的 PR 走 `ci-docs.yml` |
| msime-windows | 产品锁与输入校验、x64/Win32 TSF 构建与协议测试、内嵌 Engine 和 Server 使用产品实际数据测试、Server 可移植测试的 Linux ASan/UBSan、GUI 库边界和原生测试、设置页测试与构建、皮肤清单校验及临时目录安装测试、安装包文件与升级保留用户数据测试 |
| msime-web | 固定 Docs 子模块、冻结 pnpm 安装、Biome lint、数据生成与站点脚本测试、清单自动合并守卫回归、TypeScript/站点与静态页构建 |
| msime-cloud | 每次 push 和 PR：Admin Web lint 与提交产物一致性，契约与 OpenAPI 生成检查，Linux/Windows 上的 `go test -race`、`go vet` 和构建；PostgreSQL 用户体系集成测试；原生查询引擎配合发布词库的真实查询与 HTTP 测试、覆盖率检查和只读生产镜像冒烟 |
| msime-docs / .github | 本地 Markdown 链接、图片路径和标题锚点检查 |
| msime-dictionary | PR 上用固定 msime 提交构建的 `msime-dict-build` 检查 `custom/` 新增条目并评论结果，校验专业词包格式与读音；`main` 上由 release-please 发布 |
| msime-customdict | PR 上同样的 `words.txt` 条目检查和词包校验 |
| msime-plugins | 用固定 msime 提交构建的 `msime-pack` 检查插件包与模板并打包；`main` 的 push 更新滚动的 `packs` Release |
| Google-PinyinIME-Rev | 三平台解码库和建库工具编译、使用仓内词库验证候选输出、Linux ASan/UBSan |
| pinyin_cpp | 三平台原型编译、合成 SQLite 数据上的查询排序及分词回归 |
| pinyin_python | Python 3.12/3.14、Linux/Windows 语法检查和已有分词断言 |
| chinese-ime-lm | Python 3.12/3.14、Linux/Windows 语法检查和合成语料清洗测试；Rust 参考实现的 fmt、clippy、测试和 1.75 最低版本检查，训练管线的 ruff 与入口解析，评测集格式检查；Release 发布或编辑时校验权重附件与说明中的摘要 |
| msime-skin-example | TOML 与资源验证、Windows 临时目录中的安装、激活和重复安装测试、本地文档链接检查 |

chinese-ime-lm 的 CI 不下载完整语料，也不训练 KenLM 或神经模型；皮肤测试不安装到维护者的真实用户目录。原生 IME 焦点、跨 DPI、系统授权和签名安装仍需要发布前实际环境验证。

## 分支模型下的 CI

msime 和 msime-windows 的默认分支是 `develop`，`main` 是发布分支，分支约定的原始说明见[组织 AGENTS.md 的分支模型](https://github.com/metasequoiaime/.github/blob/main/AGENTS.md#分支模型)（该节仍按合仓前的四个代码仓描述，CI 相关部分以本节为准）。这对 CI 的含义：

- 功能 CI 和质量检查监听 `develop` 和 `main` 两条分支的 push，PR 触发不按分支过滤，因此提到 `develop` 的 PR 与从前提到 `main` 时跑的是同一组检查。两仓的 CodeQL 按每日定时在默认分支 `develop` 上运行（msime 也可手动运行），不随 push 触发。
- msime-windows 的 `release.yml` 只监听 `main` 的 push。日常合并进 `develop` 不消耗签名额度，也不产生版本号。msime 的各平台发布工作流只能手动触发，见下文。
- msime-windows 的 `release.yml` 里每一处 release-please 调用都显式写 `target-branch: main`。这个动作的默认值是仓库默认分支，改成 `develop` 之后它会把版本 PR、CHANGELOG 和 tag 全写到 `develop`，而流水线按名字等待的 `release-please--branches--main--*` 分支根本不会出现——没有报错，发布链只是停住。
- 两个代码仓各有一个 `Branch guard` 工作流（检查名 `Base branch`），在 PR 的 base 是 `main` 时检查 head：只放行 `develop`、`release/*` 和 `release-please--*`；msime 另要求 head 来自本仓库而不是 fork。它是工作流检查而不是 ruleset 规则，因为 ruleset 能保护 base 分支，但说不了哪些 head 可以指向它。工作流注释要求把它加入 `main` 的必需检查，但两仓当前的必需检查里还没有它，因此它失败时只显示为红色检查，不会单独阻止合并。
- 其余维护中的仓库（包括 msime-docs、msime-web 和 .github）默认分支都是 `main`，只用 `main`，它们的工作流不需要 `develop` 触发，也不需要 `Branch guard`。
- 健康审计按默认分支查询工作流的运行记录，因此 msime-windows 的 `release.yml` 以及 msime 的 `release-macos.yml`、`release-ios.yml` 在 `tools/health-policy.json` 里写成 `{"branch": "main", "cadence": null}`。不声明的话它们在 `develop` 上一条运行都没有，会被报成从未运行过的工作流，而发布路径恰恰是最不该失去监控的那条。

## 发布与仓库设置

Windows 正式版按 msime-windows 当前策略自动发布：把 `develop` 合进 `main` 之后，版本 PR 经过 CI 自动合并，再构建、签名和发布，发布后通过 `repository_dispatch` 通知 msime-web 刷新下载清单；`workflow_dispatch` 用于修复已有 draft。签名成本与发布节奏由产品仓库的[发布说明](https://github.com/metasequoiaime/msime-windows/blob/develop/docs/product-release.md)说明。CI 门禁改动不重定义现有 release-please 版本方案、draft 校验、产品锁或下载清单通知。

msime 不使用 release-please，也没有随 push 触发的发布。`release-macos.yml`、`release-ios.yml`、`release-linux.yml`、`release-android.yml`、`release-harmony.yml`、`release-windows.yml` 和 `release-dictionary.yml` 都只能手动触发；平台版本默认读 `platforms/<平台>/version.txt`，各平台用独立的 tag 前缀（如 `macos-v`、`linux-v`，词库为 `dict-v`）。`develop` 上的这些工作流要显式勾选 `publish` 才会创建 Release，否则只上传 workflow artifact，避免在特性分支上手动运行时误发公开版本；macOS 正式版在 DMG 通过公证校验后还会更新 `metasequoiaime/homebrew-tap` 的 cask。这些改动尚未进入 `main`：`main` 上没有 `release-dictionary.yml`，发布工作流也没有 `publish` 开关，手动运行即直接发布，macOS 产物为 tar.gz，也不更新 Homebrew tap（该 tap 目前只有 README）。

新检查首次运行通过后再加入分支 ruleset 的 required checks，使用 GitHub 实际显示的检查名；不能把没有运行过或会被路径过滤永久跳过的任务设为必需。msime 的纯文档 PR 不跑平台工作流，由 `ci-docs.yml` 以同名检查补报必需的 `macOS 15 arm64`、`macOS 15 x86_64` 和 `iOS Simulator`，正是为此。`main` 和 `develop` 都应禁止强推和删除，并要求 PR 及通过检查；msime 和 msime-windows 的 ruleset 因此显式列出这两条分支，不再写成 `~DEFAULT_BRANCH`——默认分支改成 `develop` 之后，那个占位符会把 `main` 漏在保护之外。维护者的既有 bypass 规则需保留。

维护者应启用 dependency graph、Dependabot alerts/security updates、secret scanning 和 push protection。工作流文件无法代替这些 GitHub 仓库设置。CodeQL 上传成功及 Settings 的实际状态是启用成功的证据。

msime-web 的 `sync-downloads.yml` 重新生成并测试 `public/update.json` 和 `public/platforms.json` 后，通过 `automation/downloads` 创建或更新自动 PR；它按定时运行，也接受上游发布流水线发来的 `repository_dispatch`。`sync-community.yml` 以同样方式通过 `automation/community-snapshot` 更新 `public/community.json`。main 要求 PR 以及 Build、Workflow validation、Dependency review，检查来源固定为 GitHub Actions。自动合并前再次验证同仓分支、该分支只改动它负责的清单文件、精确 head 和实时门禁；只有通过这些规则才请求 auto-merge。组织现有凭据触发正常 PR CI，合并后保持既有 Pages 部署集成。不要恢复 main 直推来绕过检查。

## 本地复现

工作流校验：安装 Go 后执行 `go install github.com/rhysd/actionlint/cmd/actionlint@v1.7.12`，在仓库根运行 `SHELLCHECK_OPTS=--severity=warning actionlint`。也要安装 ShellCheck 才能检查内嵌 shell。

文档：使用 lychee v0.24.2，运行 `lychee --offline --include-fragments --no-progress './**/*.md'`。离线门禁只检查仓内目标，外站可用性不阻断无关贡献。

各仓 `.github/workflows/` 下的功能 CI（通常是 `ci.yml`；msime 另有 `ci-macos.yml`、`ci-ios.yml`、`ci-platforms.yml`）是依赖安装与测试命令的权威来源。没有执行过的宿主、安装或发布验证不得报告为通过。

msime 的 GitHub Actions 只覆盖验证的一半：Rust workspace 的 `cargo test` 与 clippy、前端类型检查与 vitest、Wine 下的 Windows 套件等只在本地由 `bash scripts/verify-local.sh` 运行（`--quick` 只编译，作为合并前门禁），具体分工见 msime 的 [ARCHITECTURE.md](https://github.com/metasequoiaime/msime/blob/develop/ARCHITECTURE.md#验证)。

健康审计：`python -m unittest discover -s tests` 验证停跑、连续失败、取消运行、缺失权限和保护退化检测；`python tools/audit_repositories.py --health --report health-report.json` 检查实时设置，需要有权限读取公开仓库管理/安全设置的 gh 登录。

msime 的 Linux CI 只在固定容器内构建 IBus engine、Fcitx5 插件和 provider 入口并跑单元测试，不启动 D-Bus 或 IBus daemon。GTK/Qt 真实输入框冒烟测试在 `platforms/linux/tests/tools/check-container.sh` 的隔离 Xvfb 与 D-Bus 中运行，需要已验证的词库资源，目前不在 CI 中。物理多显示器 DPI、Wayland、Windows TSF 宿主、macOS 系统授权和真实签名安装同样需要实际环境验证；这些项目目前仍有平台环境限制，不能宣称已全部自动化。
