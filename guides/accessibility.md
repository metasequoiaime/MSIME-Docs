# 无障碍现状

输入法决定屏幕阅读器用户能不能打字，所以「现在做到了什么、没做到什么」应该有一份可查的说明，而不是让人装完才发现。

**这是一份现状说明，不是承诺。** 下面每一条都对着源码核对过，包括对本项目不利的部分。按 2026-09-06 的源码整理。

## 一句话

**目前没有任何针对屏幕阅读器的专门适配。** 候选窗、悬浮工具栏和托盘菜单都没有暴露无障碍信息；设置窗口有部分 ARIA 标记；iOS 宿主 App 用了 SwiftUI 的无障碍修饰符。如果你依赖屏幕阅读器输入中文，现阶段请谨慎评估。

## 各界面的实际情况

| 界面 | 实现方式 | 无障碍现状 |
| --- | --- | --- |
| 候选窗（Windows） | WebView2 承载的 HTML | **无任何 `aria-*` 或 `role` 属性**，候选项对屏幕阅读器不可见 |
| 悬浮工具栏、托盘菜单（Windows） | 同上 | 同上 |
| 设置窗口（Windows） | WebView2 承载的 HTML | 部分控件带 `aria-*`，覆盖不完整、未做系统性检查 |
| 输入本身（Windows） | TSF 文本服务 | 未做 UI Automation / IAccessible 集成 |
| macOS 前端 | InputMethodKit + AppKit | 未做 NSAccessibility 集成 |
| iOS 宿主 App | SwiftUI | 使用了 `accessibilityHint` / `accessibilityIdentifier` / `accessibilityHidden` |
| Linux 前端 | IBus + GTK | 依赖 GTK 与 IBus 各自的默认行为，本项目未额外适配 |

## 这意味着什么

在 Windows 上，输入过程本身走的是 TSF。宿主应用如何把组字状态报告给屏幕阅读器，很大程度上取决于宿主自己对 TSF 的支持程度，不完全由输入法决定。但**候选列表是输入法自己画的**——它是 WebView2 里的一段 HTML，而那段 HTML 没有任何无障碍标记。屏幕阅读器读不出「当前有哪些候选、选中的是第几个」。

这不是一个已知难题卡住了，而是还没有人做过。

## 可以先做的事

如果你想在这条线上贡献，从候选窗的 HTML 开始是成本最低、收益最直接的：

- 给候选列表加 `role="listbox"` / `role="option"` 与 `aria-selected`，让「有哪些候选、选中第几个」可被读出；
- 加 `aria-live` 区域播报页码与候选页变化；
- 设置窗口做一次系统性的 ARIA 检查，补齐缺失的标签与分组；
- 在真实的屏幕阅读器（NVDA、Narrator、VoiceOver、Orca）下实测并记录结果——**这一步没有替代品**，也不需要写代码。

相关代码在 [MSIME-Windows](https://github.com/metasequoiaime/MSIME-Windows) 的 `ui-html/webview2/candwnd/`。参与方式见[招募文档](https://github.com/metasequoiaime/.github/blob/main/RECRUITING.md)。

## 有帮助的现有功能

这些不是无障碍适配，但对部分用户确实有用：

- **候选窗字号可调**，见各平台指南的外观设置。
- **深色 / 浅色主题**跟随系统。
- **语音输入**可以完全替代键盘输入，见各平台指南的语音一节。注意它需要联网并自备 API token。
- **手写与屏幕键盘**（Windows、Linux）提供了非键盘的输入路径。

## 反馈

无障碍相关的问题按普通缺陷提交到对应平台的仓库即可，请在标题里写明「无障碍」，便于分类。如果你能提供屏幕阅读器的实际输出或录屏，价值远高于文字描述。
