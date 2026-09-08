# Linux（IBus）使用指南

水杉 Linux 前端接入 IBus，提供全拼、雙拼、五筆和日語羅馬音輸入，以及 GTK 設定與桌面工具。本文按 2026-09-06 的 Linux 原始碼整理；已發布安裝套件與主分支可能存在功能差異，請同時檢視對應 Release。

本頁採用臺灣繁體中文用語；引號內的應用程式選項、程式碼、檔名與範例保留原文，方便對照實際介面。

## 下載與啟用

在 [Linux Releases](https://github.com/metasequoiaime/MSIME-Linux/releases) 選擇適合發行版與處理器架構的 DEB 或 RPM。`amd64` / `x86_64` 對應 64 位元 Intel/AMD，`arm64` / `aarch64` 對應 64 位元 ARM。包的系統依賴以對應版本發布說明和套件管理器檢查結果為準。

Debian/Ubuntu 使用 `sudo apt install ./下载的文件.deb`；使用 DNF 的系統可用 `sudo dnf install ./下载的文件.rpm`。將示例檔名替換為實際下載路徑。不要混用發行版套件和目前使用者安裝；原始碼安裝及 TGZ 的開發用法見 [Linux 儲存庫](https://github.com/metasequoiaime/MSIME-Linux#readme)。

安裝後登出並重新登入，在桌面“输入源”或 IBus 設定中新增“Metasequoia IME”，切換後在文字框輸入 `nihao`，按空格選擇候選。本前端使用 IBus；桌面使用其他輸入法框架時需按發行版說明配置 IBus 會話。

如果是透過原始碼的 `scripts/install.sh` 為目前使用者安裝，指令碼還會寫入 `environment.d/10-metasequoiaime.conf`，讓 IBus 找到使用者元件目錄，需登出登入才生效。IBus 不會僅因詞庫位於 `XDG_DATA_HOME` 就自動發現元件；目前使用者安裝的具體註冊與臨時會話命令見實現倉 README。

## 輸入與快捷鍵

| 操作 | 方法 |
| --- | --- |
| 切換輸入方案 | IBus 語言欄選單選擇全拼、雙拼、五筆或日語羅馬音 |
| 中文／直接輸入 | 單獨輕敲任一 `Shift`；組合鍵中的 Shift 不觸發切換 |
| 選擇候選 | 空格選反白項，`1`～`9`（含小鍵盤）或滑鼠選對應項 |
| 移動反白 | `↑` / `↓` |
| 翻頁 | Page Up / Page Down、`-` / `=`、Shift + Tab / Tab；逗號句號翻頁可選 |
| 提交原始輸入／取消 | `Enter` / `Esc` |
| 中英文標點 | `Ctrl + .` |
| 半形／全形 | `Ctrl + Shift + Space` |
| 獨立英文候選 | `Ctrl + Shift + E` 進入或退出 |

單引號用於拼音分隔。啟用“以词定字”後，`[` / `]` 選反白詞的首／末漢字；如果同時啟用方括號翻頁，翻頁優先。智慧標點可在字母數字之後保留英文符號；重複標點轉中文和成對標點分別可配置。

## 設定與輔助碼

從桌面應用選單或終端啟動 `metasequoia-ime-settings`。可調整預編輯顯示、翻頁鍵、輔助碼、調頻、混合候選與線上功能。原始按鍵、拼音分詞和隱藏預編輯僅影響行內顯示，隱藏預編輯仍保留候選表。

全拼與雙拼的輔助碼分別開啟，支援藍天小雨點、自然碼、首右 2.0、首右 Plus、小鶴。輔助碼在完整拼寫之後才生效。調頻可選關閉、置頂、折半、線性前移或一次置前；觸發次數和線性步長取值為 1～10。

設定檔案為 `${XDG_CONFIG_HOME:-$HOME/.config}/metasequoiaime/config.ini`。通常優先用設定程式修改；手工編輯前先停止水杉引擎，以免執行中儲存覆蓋改動。完整配置鍵與分組見 [Linux 操作與設定](https://github.com/metasequoiaime/MSIME-Linux#操作与设置)。

## 快捷模式與混合候選

全拼／雙拼、無活動組詞時，以下快捷鍵進入單次輸入模式，相關入口可在設定中關閉：

| 快捷鍵 | 用途 |
| --- | --- |
| `Shift + U` | Unicode：輸入 `4e00` 或 `+1f600`，空格選擇，`Shift + 数字` 選其他候選 |
| `Shift + T` | 日期 `rq` / `date`、時間 `sj` / `time`、星期 `xq` / `week` |
| `Shift + K` | 按字母編碼查詢快捷片語 |
| `Shift + E` / `Shift + M` | 搜尋 Emoji／顏文字，支援拼音、簡拼、雙拼或英文關鍵詞 |
| `Shift + J` | 超級簡拼，例如 `nh`；雙拼按目前方案轉換聲母鍵 |
| `Shift + Y` / `Shift + R` | 臨時英文／日語羅馬音，提交、取消或刪空字首後返回中文方案 |

混合英文、Emoji 和顏文字候選預設關閉，可分別開啟。英文觸發長度可設為 1～8（預設 2）；Emoji 和顏文字從兩個字元起匹配。獨立英文模式與臨時英文不同：獨立模式送出文字後繼續按英文匹配，需再次按 `Ctrl + Shift + E` 退出。

## 聯網功能與語音

**雲端候選字預設開啟**，輸入空閒 500 毫秒後向 Google 傳送目前拼寫，補充第二候選位。需要關閉時，在設定中取消雲端候選字，或停止引擎後把 `config.ini` 的 `[online]` 中 `cloud-enabled` 設為 `false`。本地輸入不依賴這些請求。

AI 建議預設關閉，支援 DeepSeek、OpenAI、SiliconFlow、Groq 和自訂 HTTPS 介面；會傳送拼寫及相關上下文。候選翻譯預設啟用本地釋義，只改變展示文字；選擇 DeepLX 後才把候選傳送到所配置的 HTTPS 服務。

語音透過獨立命令執行。在設定程式開啟語音並配置服務端點、模型和憑證後：

```sh
metasequoia-ime-voice --file recording.wav
metasequoia-ime-voice --record 5
```

第一條上傳已有 WAV；第二條錄製 5 秒，需安裝 `arecord` 或 `pw-record` 並有可用音訊裝置。辨識結果列印到終端，需要自行複製到目標應用；它不是 Windows/macOS 的錄音快捷鍵送出文字流程。啟用文字潤色後，轉寫文字還會發往配置的整理介面。

AI、翻譯和語音憑證按服務商儲存在桌面 Secret Service（例如 GNOME Keyring），不寫入 `config.ini`。憑證儲存失敗時檢查桌面鑰匙串是否可用、已解鎖。備份普通配置檔案不會備份這些金鑰。

## 桌面工具

`metasequoia-ime-tools` 提供剪貼簿歷史、螢幕小鍵盤和手寫工作區；`metasequoia-ime-toolbar` 提供工具啟動入口。螢幕小鍵盤和手寫工作區可把文字送到剪貼簿，再貼上到目標應用。

剪貼簿歷史預設關閉，開啟後在本地儲存。手寫透過本機 Tesseract 辨識；缺少後端時安裝發行版的 Tesseract 與簡體中文語言包，例如 Debian/Ubuntu 的 `tesseract-ocr` 和 `tesseract-ocr-chi-sim`。

## 資料、升級與解除安裝

使用者資料預設位於 `~/.local/share/metasequoiaime/`；設定了 `XDG_DATA_HOME` 時位於該目錄的 `metasequoiaime/` 子目錄。`msime_user.db` 記錄學習權重和使用者變更。備份時停止水杉引擎，複製資料目錄與配置目錄；不要只備份隨發行包提供的基礎詞庫。

包安裝透過對應套件管理器升級或解除安裝，例如 `sudo apt remove metasequoia-ime-linux`。目前使用者原始碼安裝使用實現倉的 `scripts/install.sh` 升級，安裝前停止引擎，指令碼會驗證新資料並回放使用者記錄。

原始碼安裝的 `scripts/uninstall.sh` 保留學習資料；只有確認無需保留時才使用 `--purge`，它會進一步清理設定、詞庫、剪貼簿歷史與學習資料。解除安裝後重啟 IBus 或登出登入以重新整理輸入源。

## 故障排查與回報

- **列表沒有水杉**：區分系統套件與使用者安裝，確認目前桌面執行 IBus；使用者安裝後需讓會話載入元件路徑。
- **只有某個應用不響應**：在另一個文字編輯器對比，記錄桌面、Wayland/X11、宿主版本和所選方案。
- **語音無法錄製**：檢查錄音工具、裝置和權限；`--file` 可幫助區分採集失敗與轉寫失敗。
- **線上結果不出現**：檢查功能開關、HTTPS 地址、模型及鑰匙串，先確認本地候選正常。

向 [Linux Issues](https://github.com/metasequoiaime/MSIME-Linux/issues) 提供發行版、桌面環境、會話型別、輸入法版本、安裝方式與重現步驟。日誌和截圖先去除憑證及真實輸入。完整資料流見 [Linux 隱私說明](https://github.com/metasequoiaime/MSIME-Linux/blob/main/PRIVACY.md)。
