# 无障碍现状

输入法决定屏幕阅读器用户能不能打字，所以「现在做到了什么、没做到什么」应该有一份可查的说明，而不是让人装完才发现。

**这是一份现状说明，不是承诺。** 下面每一条都对着源码核对过，包括对本项目不利的部分。按 2026-10-01 的源码整理：Windows 对应 [msime-windows](https://github.com/metasequoiaime/msime-windows) 的 v0.9.3 发布版，macOS 对应 [msime](https://github.com/metasequoiaime/msime) 的 v0.50.0-build.9 发布版，iOS 对应 TestFlight 测试版 ios-v0.50.0-build.14，Linux 对应已归档的 msime-linux 仓库发布的 v0.8.2。

## 一句话

**Windows 版目前没有任何针对屏幕阅读器的专门适配。** 候选窗、悬浮工具栏和托盘菜单都没有暴露无障碍信息；设置窗口有部分 ARIA 标记。macOS 的候选窗、悬浮工具栏和设置窗口，以及 iOS 键盘的按键和候选，带有供 VoiceOver 朗读的无障碍标签，但实际朗读效果没有在 VoiceOver 下实测。如果你依赖屏幕阅读器输入中文，现阶段请谨慎评估。

## 各界面的实际情况

| 界面 | 实现方式 | 无障碍现状 |
| --- | --- | --- |
| 候选窗（Windows） | 默认由自研 msimeui 框架以 Direct2D 原生绘制；「设置 → 外观 → 界面渲染」可改为 WebView2 承载的 HTML | 原生绘制未做 UI Automation 集成；WebView2 版 **无任何 `aria-*` 或 `role` 属性**；两种方式下屏幕阅读器都无法从候选窗读出候选项 |
| 悬浮工具栏（Windows） | 同上 | 同上 |
| 托盘菜单（Windows） | 同上 | 原生绘制未做 UI Automation 集成；WebView2 版只给菜单图标加了 `aria-hidden`，菜单项没有 `role` 或无障碍标签 |
| 设置窗口（Windows） | WebView2 承载的 HTML | 部分控件带 `aria-*`，覆盖不完整、未做系统性检查 |
| 表情面板、屏幕键盘、手写识别板（Windows） | 自研 msimeui 框架（Win32 + Direct2D） | 未做 UI Automation 集成 |
| 输入本身（Windows） | TSF 文本服务 | 未做 UI Automation / IAccessible 集成；实现了 TSF 的 `ITfCandidateListUIElement`（供自绘候选的宿主使用），屏幕阅读器能否经此读到候选未实测 |
| macOS 前端 | InputMethodKit + AppKit | 候选按钮、翻页按钮、预编辑文本、悬浮工具栏按钮和设置窗口控件带 `accessibilityLabel`；开启「切换中英文时显示提示」（默认开启）时，切换中英文会请求 VoiceOver 播报「中文输入」或「英文输入」；候选按钮没有标出哪一项被高亮，高亮变化时也没有主动播报 |
| iOS 键盘与宿主 App | UIKit 键盘扩展 + SwiftUI 宿主 App | 按键、候选（含序号与释义）和各面板带 `accessibilityLabel` / `accessibilityValue`，打开面板时发出 VoiceOver 屏幕变化通知；宿主 App 使用了 `accessibilityLabel` / `accessibilityHint` / `accessibilityIdentifier` / `accessibilityHidden` 等修饰符 |
| Linux 前端 | IBus + GTK | 依赖 GTK 与 IBus 各自的默认行为，本项目未额外适配。msime 开发分支新增的 Fcitx5 入口尚未发布，同样未额外适配 |

## 这意味着什么

在 Windows 上，输入过程本身走的是 TSF。宿主应用如何把组字状态报告给屏幕阅读器，很大程度上取决于宿主自己对 TSF 的支持程度，不完全由输入法决定。但**候选列表是输入法自己画的**——默认由 Direct2D 原生绘制，没有 UI Automation 支持；改用 WebView2 渲染时是一段 HTML，同样没有任何无障碍标记。屏幕阅读器无法从候选窗本身读出「当前有哪些候选、选中的是第几个」；经 TSF 的 `ITfCandidateListUIElement` 能否读到，尚未实测。

这不是一个已知难题卡住了，而是还没有人做过。

## 可以先做的事

如果你想在这条线上贡献，从候选窗开始是收益最直接的：

- 给默认的 Direct2D 候选窗（msimeui 的 `CandidateList`）实现 UI Automation provider，暴露候选列表与选中项；
- WebView2 渲染时，给候选列表加 `role="listbox"` / `role="option"` 与 `aria-selected`，让「有哪些候选、选中第几个」可被读出；
- WebView2 渲染时，加 `aria-live` 区域播报页码与候选页变化；
- 设置窗口做一次系统性的 ARIA 检查，补齐缺失的标签与分组；
- 在真实的屏幕阅读器（NVDA、Narrator、VoiceOver、Orca）下实测并记录结果——**这一步没有替代品**，也不需要写代码。

相关代码在 [msime-windows](https://github.com/metasequoiaime/msime-windows) 的 `ui/src/CandidateList.cpp`（默认的原生候选窗控件）、`server/src/window/candidate_presenter.cpp`，以及 WebView2 版的 `ui-html/webview2/candwnd/`。参与方式见[招募文档](https://github.com/metasequoiaime/.github/blob/main/RECRUITING.md)。

## 有帮助的现有功能

这些不是无障碍适配，但对部分用户确实有用：

- **候选窗字号可调**，见各平台指南的外观设置。
- **深色 / 浅色主题**跟随系统。
- **语音输入**可以完全替代键盘输入，见各平台指南的语音一节。Windows 与 Linux 上需要联网并自备 API token；macOS 还可选择本地 Whisper 模型（模型文件需自行准备），但开启文本整理后转写文本仍会联网发送。
- **手写与屏幕键盘**（Windows、Linux）提供了非键盘的输入路径。

## 反馈

无障碍相关的问题按普通缺陷提交即可：Windows 提交到 [msime-windows](https://github.com/metasequoiaime/msime-windows/issues)，macOS、iOS 与 Linux 提交到 [msime](https://github.com/metasequoiaime/msime/issues)。请在标题里写明「无障碍」，便于分类。如果你能提供屏幕阅读器的实际输出或录屏，价值远高于文字描述。
