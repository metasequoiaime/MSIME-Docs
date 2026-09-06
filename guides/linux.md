# Linux（IBus）使用指南

水杉 Linux 前端接入 IBus，提供全拼、双拼、五笔和日语罗马音输入，以及 GTK 设置与桌面工具。本文按 2026-09-06 的 Linux 源码整理；已发布安装包与主分支可能存在功能差异，请同时查看对应 Release。

## 下载与启用

在 [Linux Releases](https://github.com/metasequoiaime/MSIME-Linux/releases) 选择适合发行版与处理器架构的 DEB 或 RPM。`amd64` / `x86_64` 对应 64 位 Intel/AMD，`arm64` / `aarch64` 对应 64 位 ARM。包的系统依赖以对应版本发布说明和包管理器检查结果为准。

Debian/Ubuntu 使用 `sudo apt install ./下载的文件.deb`；使用 DNF 的系统可用 `sudo dnf install ./下载的文件.rpm`。将示例文件名替换为实际下载路径。不要混用发行版包和当前用户安装；源码安装及 TGZ 的开发用法见 [Linux 仓库](https://github.com/metasequoiaime/MSIME-Linux#readme)。

安装后注销并重新登录，在桌面“输入源”或 IBus 设置中添加“Metasequoia IME”，切换后在文本框输入 `nihao`，按空格选择候选。本前端使用 IBus；桌面使用其他输入法框架时需按发行版说明配置 IBus 会话。

如果是通过源码的 `scripts/install.sh` 为当前用户安装，脚本还会写入 `environment.d/10-metasequoiaime.conf`，让 IBus 找到用户组件目录，需注销登录才生效。IBus 不会仅因词库位于 `XDG_DATA_HOME` 就自动发现组件；当前用户安装的具体注册与临时会话命令见实现仓 README。

## 输入与快捷键

| 操作 | 方法 |
| --- | --- |
| 切换输入方案 | IBus 语言栏菜单选择全拼、双拼、五笔或日语罗马音 |
| 中文／直接输入 | 单独轻敲任一 `Shift`；组合键中的 Shift 不触发切换 |
| 选择候选 | 空格选高亮项，`1`～`9`（含小键盘）或鼠标选对应项 |
| 移动高亮 | `↑` / `↓` |
| 翻页 | Page Up / Page Down、`-` / `=`、Shift + Tab / Tab；逗号句号翻页可选 |
| 提交原始输入／取消 | `Enter` / `Esc` |
| 中英文标点 | `Ctrl + .` |
| 半角／全角 | `Ctrl + Shift + Space` |
| 独立英文候选 | `Ctrl + Shift + E` 进入或退出 |

单引号用于拼音分隔。启用“以词定字”后，`[` / `]` 选高亮词的首／末汉字；如果同时启用方括号翻页，翻页优先。智能标点可在字母数字之后保留英文符号；重复标点转中文和成对标点分别可配置。

## 设置与辅助码

从桌面应用菜单或终端启动 `metasequoia-ime-settings`。可调整预编辑显示、翻页键、辅助码、调频、混合候选与在线功能。原始按键、拼音分词和隐藏预编辑仅影响行内显示，隐藏预编辑仍保留候选表。

全拼与双拼的辅助码分别开启，支持蓝天小雨点、自然码、首右 2.0、首右 Plus、小鹤。辅助码在完整拼写之后才生效。调频可选关闭、置顶、折半、线性前移或一次置前；触发次数和线性步长取值为 1～10。

设置文件为 `${XDG_CONFIG_HOME:-$HOME/.config}/metasequoiaime/config.ini`。通常优先用设置程序修改；手工编辑前先停止水杉引擎，以免运行中保存覆盖改动。完整配置键与分组见 [Linux 操作与设置](https://github.com/metasequoiaime/MSIME-Linux#操作与设置)。

## 快捷模式与混合候选

全拼／双拼、无活动组词时，以下快捷键进入单次输入模式，相关入口可在设置中关闭：

| 快捷键 | 用途 |
| --- | --- |
| `Shift + U` | Unicode：输入 `4e00` 或 `+1f600`，空格选择，`Shift + 数字` 选其他候选 |
| `Shift + T` | 日期 `rq` / `date`、时间 `sj` / `time`、星期 `xq` / `week` |
| `Shift + K` | 按字母编码查询快捷短语 |
| `Shift + E` / `Shift + M` | 搜索 Emoji／颜文字，支持拼音、简拼、双拼或英文关键词 |
| `Shift + J` | 超级简拼，例如 `nh`；双拼按当前方案转换声母键 |
| `Shift + Y` / `Shift + R` | 临时英文／日语罗马音，提交、取消或删空前缀后返回中文方案 |

混合英文、Emoji 和颜文字候选默认关闭，可分别开启。英文触发长度可设为 1～8（默认 2）；Emoji 和颜文字从两个字符起匹配。独立英文模式与临时英文不同：独立模式上屏后继续按英文匹配，需再次按 `Ctrl + Shift + E` 退出。

## 联网功能与语音

**云候选默认开启**，输入空闲 500 毫秒后向 Google 发送当前拼写，补充第二候选位。需要关闭时，在设置中取消云候选，或停止引擎后把 `config.ini` 的 `[online]` 中 `cloud-enabled` 设为 `false`。本地输入不依赖这些请求。

AI 建议默认关闭，支持 DeepSeek、OpenAI、SiliconFlow、Groq 和自定义 HTTPS 接口；会发送拼写及相关上下文。候选翻译默认启用本地释义，只改变展示文字；选择 DeepLX 后才把候选发送到所配置的 HTTPS 服务。

语音通过独立命令运行。在设置程序开启语音并配置服务端点、模型和凭据后：

```sh
metasequoia-ime-voice --file recording.wav
metasequoia-ime-voice --record 5
```

第一条上传已有 WAV；第二条录制 5 秒，需安装 `arecord` 或 `pw-record` 并有可用音频设备。识别结果打印到终端，需要自行复制到目标应用；它不是 Windows/macOS 的录音快捷键上屏流程。启用文本润色后，转写文本还会发往配置的整理接口。

AI、翻译和语音凭据按服务商保存在桌面 Secret Service（例如 GNOME Keyring），不写入 `config.ini`。凭据保存失败时检查桌面钥匙串是否可用、已解锁。备份普通配置文件不会备份这些密钥。

## 桌面工具

`metasequoia-ime-tools` 提供剪贴板历史、屏幕键盘和手写工作区；`metasequoia-ime-toolbar` 提供工具启动入口。屏幕键盘和手写工作区可把文本送到剪贴板，再粘贴到目标应用。

剪贴板历史默认关闭，开启后在本地保存。手写通过本机 Tesseract 识别；缺少后端时安装发行版的 Tesseract 与简体中文语言包，例如 Debian/Ubuntu 的 `tesseract-ocr` 和 `tesseract-ocr-chi-sim`。

## 数据、升级与卸载

用户数据默认位于 `~/.local/share/metasequoiaime/`；设置了 `XDG_DATA_HOME` 时位于该目录的 `metasequoiaime/` 子目录。`msime_user.db` 记录学习权重和用户变更。备份时停止水杉引擎，复制数据目录与配置目录；不要只备份随发行包提供的基础词库。

包安装通过对应包管理器升级或卸载，例如 `sudo apt remove metasequoia-ime-linux`。当前用户源码安装使用实现仓的 `scripts/install.sh` 升级，安装前停止引擎，脚本会验证新数据并回放用户记录。

源码安装的 `scripts/uninstall.sh` 保留学习数据；只有确认无需保留时才使用 `--purge`，它会进一步清理设置、词库、剪贴板历史与学习数据。卸载后重启 IBus 或注销登录以刷新输入源。

## 故障排查与反馈

- **列表没有水杉**：区分系统包与用户安装，确认当前桌面运行 IBus；用户安装后需让会话加载组件路径。
- **只有某个应用不响应**：在另一个文本编辑器对比，记录桌面、Wayland/X11、宿主版本和所选方案。
- **语音无法录制**：检查录音工具、设备和权限；`--file` 可帮助区分采集失败与转写失败。
- **在线结果不出现**：检查功能开关、HTTPS 地址、模型及钥匙串，先确认本地候选正常。

向 [Linux Issues](https://github.com/metasequoiaime/MSIME-Linux/issues) 提供发行版、桌面环境、会话类型、输入法版本、安装方式与复现步骤。日志和截图先去除凭据及真实输入。完整数据流见 [Linux 隐私说明](https://github.com/metasequoiaime/MSIME-Linux/blob/main/PRIVACY.md)。
