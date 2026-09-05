# Metasequoia IME(水杉输入法)

水杉输入法起初是一个 Windows 上的纯 TSF 输入法，现在各平台前端共用同一套 C++ 输入引擎：Windows 公开内测中，macOS 已发布，Linux（IBus）和 iOS 前端正在开发中。

## 开源代码

平台前端：

- Windows TSF 端: <https://github.com/metasequoiaime/MSIME-Windows>
- Apple 平台（macOS / iOS）: <https://github.com/metasequoiaime/MSIME-Apple>
- Linux（IBus）: <https://github.com/metasequoiaime/MSIME-Linux>

引擎与数据：

- 输入法引擎: <https://github.com/metasequoiaime/MSIME-Engine>
- Server 端: <https://github.com/metasequoiaime/MSIME-Server>
- 输入法词典: <https://github.com/metasequoiaime/MSIME-Dict>
- 个人自定义词典: <https://github.com/metasequoiaime/MSIME-CustomDict>
- 辅助码: <https://github.com/metasequoiaime/MetasequoiaImeHelpCode>
- n-gram 拼音联想算法: <https://github.com/metasequoiaime/Metasequoia-n-gram>
- 原安卓谷歌拼音输入法引擎: <https://github.com/metasequoiaime/Google-PinyinIME-Rev>

界面与工具：

- 原生 GUI 框架: <https://github.com/metasequoiaime/MSIME-UI>
- UI 界面（WebView2 资源）: <https://github.com/metasequoiaime/MetasequoiaImeUiHtml>
- 语音输入法: <https://github.com/metasequoiaime/MetasequoiaVoiceInput>
- 输入法日志: <https://github.com/metasequoiaime/MetasequoiaImeLog>
- 安装器: <https://github.com/metasequoiaime/msime-installer>
- 皮肤示例: <https://github.com/metasequoiaime/metasequoia-ime-skin-example>

## 截图

![](https://i.imgur.com/THBuxl3.png)

![](https://i.imgur.com/rOCyAic.png)

![](https://i.imgur.com/ayArNvd.png)

双拼初学者可以使用以下这个带有键位提示的皮肤：

![](https://i.imgur.com/Sq1OKRM.png)

## 特性

## 路线图

## 手动构建

## 贡献

欢迎参与。方向不限于写代码——整理词库、补文档、做本地化、测兼容性、录教程同样算贡献。可参与的方向按类别列在[招募开源开发者](https://github.com/metasequoiaime/.github/blob/main/RECRUITING.md)，通用约定见[贡献指南](https://github.com/metasequoiaime/.github/blob/main/CONTRIBUTING.md)。

开源的一个理由是隐私：输入法能看到用户输入的一切，这件事不该靠承诺保证，而该能被任何人直接读代码检查。

## 感谢

- 开源签名证书(50 欧元/年的 Sign in Cloud 版本): <https://www.certum.eu/en/>

## 许可协议

GPL-3.0.
