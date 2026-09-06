# 维护文档

本仓维护用户指南和跨平台产品架构。通用流程遵循[组织贡献指南](https://github.com/metasequoiaime/.github/blob/main/CONTRIBUTING.md)，仓库边界见 [AGENTS.md](AGENTS.md)。

## 内容放在哪里

| 内容 | 位置 |
| --- | --- |
| 下载后的安装、输入、设置、备份和排障 | `guides/` 中对应平台指南 |
| macOS 语音配置与权限 | `guides/macos-voice.md`，由 macOS 指南链接 |
| 公共职责、输入链路与数据来源 | `architecture/repositories.md` |
| 平台固定版本、公共能力接入与迁移验收 | `architecture/platform-adoption.md` |
| 阅读入口、平台范围、开发入口 | `README.md` |
| 构建命令、API 与模块测试 | 对应实现仓库 |
| 网站路由、页面样式、下载元数据 | MSIME-Web |

## 修改前核对

1. 从实际问题或用户操作出发，核对实现仓的已提交代码、设置页面和 Release。注明主分支与已发布版本的差别，不把计划中的功能当成可用功能。
2. 默认分支更新后再建立工作分支。相邻仓库有未提交改动时，不把其他会话的改动当作已发布事实。
3. 快捷键核对触发条件和冲突；导入格式核对解析器；安装与删除命令核对实际脚本。涉及清除数据时说明具体影响。
4. 示例只用虚构数据，不写令牌、真实输入或个人信息。联网功能说明发送的数据和关闭入口。

## 写作与校验

采用中文 Markdown，命令和字段用行内代码，示例必须能明确区分占位符与可直接复制内容。导入文件的代码块只放数据行，用真正的 Tab 分隔，不放表头或解释文字。

本仓没有应用构建目标。修改完成后至少运行 `git diff --check`，检查相对链接的文件与标题存在，检查 Markdown 表格和代码块渲染，并人工逐条核对变更的功能说明。PR 中写清实际核对的源码版本、执行的命令，以及未进行的系统实测；仅渲染通过不能证明安装或输入行为正确。

Windows 正文还会在官网渲染。Web 当前为二、三级标题生成目录；保持单个一级标题，不在正文依赖原始 HTML 或只在 GitHub 生效的组件。指向其他文件时，Windows 指南优先使用完整 GitHub 链接，避免被网站解释成站内相对路径。改动正文后可使用 Web 同版本的 `markdown-it` 检查表格、代码块和链接，不必修改其已固定的子模块来做内容校验。

Docs 合并后，再让 Web 的 `vendor/MSIME-Docs` 指向已合并提交并验证页面；不要在 Web 复制一份可独立编辑的指南。

## 本轮内容核对来源

2026-09-06 补全文档时核对了以下提交。它们是内容依据，不代表安装包均包含这些提交；平台指南仍以用户安装版本的 Release 为准。

| 仓库与提交 | 主要核对位置 |
| --- | --- |
| [Windows f7b3a66](https://github.com/metasequoiaime/MSIME-Windows/tree/f7b3a66b8bf1f268b165dc539f7c3a6f2381facb) | README、`ui-html/webview2/settings/ime-settings/src/partials/`、`server/src/settings/dictionary_manager.cpp`、安装说明 |
| [Apple 9c1ac5e](https://github.com/metasequoiaime/MSIME-Apple/tree/9c1ac5e0d6054f06c97fa461d9378cc0a440bde7) | README、PRIVACY、macOS 卸载脚本 |
| [Linux 484d24e](https://github.com/metasequoiaime/MSIME-Linux/tree/484d24e504c48e920445a3b43acd92814ed54672) | README、PRIVACY、`src/VoiceInputCli.cpp`、配置实现 |
| [Web 1ddeeef](https://github.com/metasequoiaime/MSIME-Web/tree/1ddeeefad32a0113e387bbc530edbb2a89ce0ad1) | README、`src/docs.ts`、`src/content-page.ts` |

架构依据同时包括[组织约定](https://github.com/metasequoiaime/.github/blob/main/AGENTS.md)。后续功能变更应更新对应正文及受影响的核对依据。
