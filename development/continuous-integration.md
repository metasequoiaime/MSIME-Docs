# 持续集成与自动维护

本文说明组织级 CI 的使用与排障；[工作流](https://github.com/metasequoiaime/.github/tree/main/.github/workflows)、[审计脚本](https://github.com/metasequoiaime/.github/blob/main/tools/audit_repositories.py)和[健康策略](https://github.com/metasequoiaime/.github/blob/main/tools/health-policy.json)仍在 `.github` 仓库维护。以下组织审计命令均从 `.github` 仓库根目录执行。

本规范覆盖组织当前维护的仓库。已归档的旧组件保留历史工作流，Windows 产品和 Engine 合仓后的目录继续由产品 CI 验证。

## 所有维护仓库的共同检查

- `Repository quality` 在 PR、main push、合并队列和手动运行时使用 actionlint 检查工作流及嵌入 Shell。ShellCheck 的 warning/error 阻断合并，风格建议不作为门禁。
- PR 的 Dependency review 拒绝新增 high/critical 已知依赖漏洞；它依赖 GitHub dependency graph，不能替代原生库构建测试。
- `CodeQL` 在 PR、main push 和每周定时扫描实际语言及 Actions。C/C++ 使用源码分析；Apple 的 Swift 通过未签名 iOS 模拟器编译提取。C/C++ 源码分析不能替代三平台编译或发现所有宏配置下的问题。
- Actions 使用完整提交 SHA；Dependabot 通过 PR 更新。依赖更新须通过相同的构建测试，不能仅凭机器人身份绕过门禁。
- CI 默认只读、使用托管 runner，设定超时并取消过时的同事件运行。发布流程按其实际用途单独保留写权限。

`Organization CI coverage` 每日检查组织当前所有未归档公开仓库的默认分支，发现缺失的功能 CI、质量检查、CodeQL、Dependabot 配置或被停用的工作流。可以手动运行，或用 `python tools/audit_repositories.py` 在已登录 gh CLI 的环境复现。PR 中运行只读覆盖检查及健康审计回归；已审查的 main 上另用组织现有 `RELEASE_PLEASE_TOKEN` 执行只读健康检查（所有 API 请求都是 GET），避免把管理权限交给 PR 代码。

健康审计按 `tools/health-policy.json` 检查各仓实际必需任务、检查结果的 GitHub Actions 来源、禁止强推/删除、PR 规则、安全扫描、依赖图、漏洞提醒和私密漏洞报告。它也检测最近三次已完成运行连续失败、排队/运行超过三小时、工作流被停用或创建一天后仍没有运行。取消的过期运行不计为失败。定时 CodeQL 十天内须成功，Web 更新清单六小时内须成功，组织审计两天内须成功；只随 push 运行的仓库在没有改动时不会因闲置误报。

失败会使 Actions 任务变红，完整发现列表作为 `repository-health` JSON artifact 保留 30 天，由 GitHub 现有通知设置通知维护者。审计不自动关闭安全功能、改规则或重跑发布。新增活跃公开仓库会要求维护者补充健康策略，已归档和私有仓库不纳入。

## 各仓库的功能验证

| 仓库 | 自动验证 |
| --- | --- |
| MSIME-Engine | Linux/macOS/Windows 引擎测试、Linux ASan/UBSan、共享协议生成检查、完整词库和移动词库消费测试、三平台语音测试 |
| MSIME-Windows | 产品锁、x86/x64 TSF、真实数据 Server 测试、GUI 边界和原生测试、设置页构建、安装包文件测试 |
| MSIME-Apple | Intel/ARM macOS 构建和测试、bundle 检查、iOS 模拟器构建和 onboarding UI 测试 |
| MSIME-Linux | Ubuntu 24.04/26.04 和 amd64/arm64、产品锁、IBus 会话、GTK 实际输入框焦点隔离及 1×/2× 缩放、用户安装和包安装冒烟 |
| MSIME-Web | 固定 Docs 子模块、冻结 pnpm 安装、TypeScript/站点构建、更新元数据回归 |
| MSIME-Docs / .github | 本地 Markdown 链接、图片路径和标题锚点检查 |
| Google-PinyinIME-Rev | 三平台解码库和建库工具编译、使用仓内词库验证候选输出 |
| pinyin_cpp | 三平台原型编译、合成 SQLite 数据上的查询排序及分词回归 |
| pinyin_python | Python 3.12/3.14、Linux/Windows 语法检查和已有分词断言 |
| Metasequoia-n-gram | Python 3.12/3.14、Linux/Windows 语法检查、合成语料预处理和文件选择测试 |
| metasequoia-ime-skin-example | TOML 与资源验证、Windows 临时目录中的安装、激活和重复安装测试 |

语料仓的 CI 不下载完整语料或训练 KenLM；皮肤测试不安装到维护者的真实用户目录。原生 IME 焦点、跨 DPI、系统授权和签名安装仍需要发布前实际环境验证。

## 发布与仓库设置

Windows 按产品仓库当前策略自动发布：版本 PR 经过 CI 后自动合并，再构建、签名和发布；`workflow_dispatch` 用于修复已有 draft。签名成本与发布节奏由产品仓库的[发布说明](https://github.com/metasequoiaime/MSIME-Windows/blob/main/docs/product-release.md)说明。CI 门禁改动不重定义现有 release-please 版本方案、draft 校验、产品锁或 Cloudflare Pages 集成。

新检查首次运行通过后再加入分支 ruleset 的 required checks，使用 GitHub 实际显示的检查名；不能把没有运行过或会被路径过滤永久跳过的任务设为必需。main 应禁止强推和删除，并要求 PR 及通过检查；维护者的既有 bypass 规则需保留。

维护者应启用 dependency graph、Dependabot alerts/security updates、secret scanning 和 push protection。工作流文件无法代替这些 GitHub 仓库设置。CodeQL 上传成功及 Settings 的实际状态是启用成功的证据。

MSIME-Apple 保留现有发布提交快进路径：在机器人 PR 无法创建时，严格核对版本提交、作者、改动文件和 main 状态，再等待手动 CI 成功后推进 main。因此保留它现有的状态检查门禁，不另加会堵住该路径的必须 PR 规则；新增质量检查也随该手动 CI 执行。

MSIME-Web 的更新清单任务生成并测试 `public/update.json` 后，通过 `automation/update-manifest` 创建或更新自动 PR。main 要求 PR 以及 Build、Workflow validation、Dependency review，检查来源固定为 GitHub Actions。自动合并前再次验证同仓分支、唯一改动文件、精确 head 和实时门禁；只有通过这些规则才请求 auto-merge。组织现有凭据触发正常 PR CI，合并后保持既有 Pages 部署集成。不要恢复 main 直推来绕过检查。

## 本地复现

工作流校验：安装 Go 后执行 `go install github.com/rhysd/actionlint/cmd/actionlint@v1.7.12`，在仓库根运行 `SHELLCHECK_OPTS=--severity=warning actionlint`。也要安装 ShellCheck 才能检查内嵌 shell。

文档：使用 lychee v0.24.2，运行 `lychee --offline --include-fragments --no-progress './**/*.md'`。离线门禁只检查仓内目标，外站可用性不阻断无关贡献。

各仓的 `.github/workflows/ci.yml` 是依赖安装与测试命令的权威来源。没有执行过的宿主、安装或发布验证不得报告为通过。

健康审计：`python -m unittest discover -s tests` 验证停跑、连续失败、取消运行、缺失权限和保护退化检测；`python tools/audit_repositories.py --health --report health-report.json` 检查实时设置，需要有权限读取公开仓库管理/安全设置的 gh 登录。

Linux GTK 自动测试在隔离 Xvfb 和 D-Bus 中通过真实键盘事件运行，覆盖正常/双倍显示缩放与输入框焦点隔离。它不代替物理多显示器 DPI、Wayland、Windows TSF 宿主、macOS 系统授权和真实签名安装的验证；这些项目目前仍有平台环境限制，不能宣称已全部自动化。
