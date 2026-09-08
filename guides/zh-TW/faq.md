# 常見問題 Q&A

從實際回報中整理的排查辦法。先找到與你相同的現象，再按步驟檢查；這裡以 Windows 為主，其他平臺請查閱對應指南。

本頁採用臺灣繁體中文用語；引號內的應用程式選項、程式碼、檔名與範例保留原文，方便對照實際介面。

## 字型與顯示

### 打字時漢字顯示成方框，應該安裝什麼字型？

先區分是**候選文字變成方框**，還是只有懸浮工具列的圖示變成方框。漢字缺字時，優先檢查“设置 → 外观 → 候选窗中文补充字体”：選擇系統中已安裝、能顯示這些漢字的中文字型；若使用的字型未安裝，先安裝對應字型，再重新開啟輸入法選擇它。主字型主要用於西文，不能代替中文補充字型。

如果只有少數生僻字顯示方框，需要覆蓋這些字的字型，換一個不含這些字的字型仍不會解決。如果候選字視窗正常、將文字送到某個軟體才變成方框，則應檢查那個軟體的字型設定。排查後仍有問題，請附具體字元、所選字型及截圖。

這是一條字型排查建議，不能憑方框現象認定是同一個 Bug。設定項依據：[Windows 指南 · 候選字視窗樣式](https://msime.app/zh-TW/docs/windows/#候選字視窗樣式)。工具列圖示問題見下一問。

### Windows 10 懸浮工具列圖示變成方框或空白，但按鈕仍能點選？

**有臨時處理辦法，修復仍在跟進。** Windows 10、v0.5.4 的回報中，維護者確認 Direct2D 工具列使用了系統未自帶的 `Segoe Fluent Icons` 圖示字型。這與候選漢字的字型不同。

可在“设置 → 外观”把“界面渲染”改為 **WebView2**，儲存後按 `Ctrl + Shift + Alt + T` 退出輸入法服務，再從開始選單啟動水杉輸入法。維護者將此作為臨時緩解辦法。切換渲染方式後需重新啟動才生效。

若要補齊字型，可從微軟官方的 [Segoe Fluent Icons 說明頁](https://learn.microsoft.com/en-us/windows/apps/design/iconography/segoe-fluent-icons-font#how-do-i-get-this-font)進入 Design resources 下載並安裝，再重啟輸入法。Windows 11 自帶此字型；微軟也提示獨立下載版可能缺少較新的圖示，因此安裝字型不保證解決所有圖示缺失。

來源：[Windows #232](https://github.com/metasequoiaime/MSIME-Windows/issues/232)，2026-09-08 核對時仍為開放狀態。

### 候選字視窗太大、字太小，或每頁候選數量不合習慣？

在“设置 → 外观”調整“候选窗字号”“候选窗预编辑字号”和“每页候选项数量”。文件所核對的介面支援字級 12～32 像素、每頁 3～9 項；也能切換橫向或縱向排列。候選數量只影響分頁，不會減少總候選結果。

如果調整後仍有文字遮擋，請記錄縮放比例、解析度、渲染方式和截圖，作為顯示問題回報，不要反覆刪除詞庫。

來源：[已關閉的 Windows #15](https://github.com/metasequoiaime/MSIME-Windows/issues/15)、[外觀設定指南](https://msime.app/zh-TW/docs/windows/#外觀)。

## 安裝與啟動

### 安裝後切換不了輸入法，Server 反覆退出，或設定視窗一閃即關？

先檢查 **Microsoft Visual C++ x64 執行階段**。Server 和設定程式是 64 位元程式，只安裝 x86 版本並不夠。在[微軟官方下載頁](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist)選擇 x64 的 `vc_redist.x64.exe` 安裝；已有安裝可嘗試“修复”，然後重啟 Windows。

仍有問題時，先儲存錯誤提示和事件檢視器記錄。不要先清空使用者資料，否則可能丟失詞條和排查依據。

依據：[Windows 指南 · 安裝後無法使用或設定視窗閃退](https://msime.app/zh-TW/docs/windows/#安裝後無法使用或設定視窗閃退)。

### 安裝好了，但找不到設定入口？

先按 `Win + Space` 切換到水杉輸入法，再右鍵語言欄的輸入法圖示，或使用懸浮工具列裡的設定入口。

首次安裝後看不到狀態圖示，可先重啟 Windows，再切換到水杉。#30 的回報者確認重啟後圖示恢復；這是該回報的有效辦法，不代表所有“设置打不开”都由同一原因造成。若視窗能出現但隨即退出，請按上一問檢查執行階段。

來源：[已關閉的 Windows #30](https://github.com/metasequoiaime/MSIME-Windows/issues/30)。

### 候選字視窗或設定頁空白，和漢字顯示方框一樣嗎？

兩種現象應分開排查。**視窗整體空白或不出現**時，先確認 Metasequoia IME Server 正在執行，再檢查 Microsoft Edge WebView2 Runtime，並對照所裝版本的發布說明。**視窗中有候選位置、但字元是方框**時，優先檢查字型。

如果只有某個應用無法顯示或送出文字，先在記事本中對比，回報時寫清應用名稱、版本及是否以管理員身份執行。

依據：[Windows 指南 · 更新、備份與故障排查](https://msime.app/zh-TW/docs/windows/#更新備份與故障排查)。

### 設定頁提示找不到 imesettings 的伺服器 IP，應該改 DNS 嗎？

不要把 `imesettings` 當作需要訪問的公網網站。#94 報告的是 v0.0.9.2 設定頁的本地資源載入失敗：開關代理均出現錯誤，維護者當時僅初步判斷與中文路徑有關，並未在討論中給出經確認的通用修復步驟。

先檢查安裝是否完整，按對應 Release 更新；若仍重現，記錄安裝路徑是否含中文、系統與輸入法版本、代理開啟狀態和錯誤頁截圖。不要照搬未知的 hosts 或 DNS 修改方案。設定整體空白時也可參考上一問檢查 WebView2。

來源：[已關閉的 Windows #94](https://github.com/metasequoiaime/MSIME-Windows/issues/94)。關閉不等於已驗證所有環境恢復正常。

## 輸入與快捷鍵

### 全拼輸入 fangan，為什麼先顯示 fan'gan，而不是“方案”？

這是拼音切分歧義。可以明確輸入 `fang'an`：先打 `fang`，再打半形單引號 `'`，再打 `an`。

維護者在 #41 說明，0.0.9.2+ 已加入相應備選，“方案”應出現在候選中，但首選切分仍可能是 `fan'gan`，並非把所有歧義都改為最長匹配。若新版仍沒有“方案”，回報完整輸入字串、版本和候選截圖。

來源：[已關閉的 Windows #41](https://github.com/metasequoiaime/MSIME-Windows/issues/41)、[手動分詞 #18](https://github.com/metasequoiaime/MSIME-Windows/issues/18)。

### “先”等常用字突然不見了，要刪除使用者詞庫嗎？

先更新並複測，不要直接刪除使用者詞庫。#36 的歷史問題中，解除安裝重灌一度恢復，但幾天後再次出現；維護者後來依據回報定位問題，並建議更新到 v0.0.9。

若目前版本仍出現，記錄輸入編碼、缺失的字、實際候選及最近進行過的詞庫操作。需要重置前，先按指南匯出使用者詞庫並備份本機資料。使用者詞庫可能含個人輸入，不要直接上傳到公開 Issue。

來源：[已關閉的 Windows #36](https://github.com/metasequoiaime/MSIME-Windows/issues/36)、[備份指南](https://msime.app/zh-TW/docs/windows/#更新備份與故障排查)。

### 英文單詞置頂後，為什麼還是排在中文後面？

先確認目前處於**中文模式下的中英混輸**，還是**獨立英文候選模式**。中文混輸中，英文候選預設不搶佔中文首位；可以用 `Ctrl + Shift + E` 進入獨立英文候選模式，再比較同一個詞的排序。

#110 中回報者還指出“手动置顶仍不能成为首位”。這部分不能簡單解釋為使用者沒開置頂，也不能因為 Issue 已關閉就認定已修復。若仍重現，請附輸入編碼、希望置頂的詞、實際順序和目前模式。

來源：[已關閉的 Windows #110](https://github.com/metasequoiaime/MSIME-Windows/issues/110)、[Windows 輸入指南](https://msime.app/zh-TW/docs/windows/#輸入)。

### Git Bash 裡按 Shift 不能切換中英文？

先確認終端宿主。#32 的修復針對 `mintty.exe`，維護者建議在 0.0.8 及以上版本複測。如果 Git Bash 執行在 Windows Terminal 中，則不屬於這條修復覆蓋的宿主。

仍失效時，寫清是獨立 Git Bash / mintty，還是 Windows Terminal 中的 Git Bash，並附版本。可在“设置 → 快捷键 → 中英文切换”選擇其他已支援的切換鍵進行對比。

來源：[已關閉的 Windows #32](https://github.com/metasequoiaime/MSIME-Windows/issues/32)。

### 中英文狀態總跳回去，或切換快捷鍵和別的軟體衝突？

先檢查“设置 → 快捷键 → 中英文切换”中啟用的 `Shift`、`Ctrl`、`Ctrl + Alt + Space`，關閉衝突的項，再對比測試。`Ctrl + Space` 由 Windows 管理，需要在 Windows 的“输入语言热键”中修改。

也檢查是否安裝了其他中英文自動切換或鍵盤增強工具。#16 的一位回報者發現與 Capsense 衝突，關閉後恢復；這只是已確認的一個衝突案例，不代表所有狀態跳變都由該工具造成。

來源：[已關閉的 Windows #16](https://github.com/metasequoiaime/MSIME-Windows/issues/16)、[快捷鍵指南](https://msime.app/zh-TW/docs/windows/#快捷鍵)。

### 可以用微軟雙拼，或一直輸出英文標點嗎？

可以。在“设置 → 输入”選擇雙拼及微軟雙拼方案；“固定标点状态”可啟用“始终使用英文标点”。它與“始终使用中文标点”互斥，啟用後不會隨中英文輸入模式自動改變標點狀態。

如果安裝版本沒有這些選項，請先對照發布說明更新。微軟雙拼已經包含在維護者確認的支援列表中，無需手動編輯方案表。

來源：[已關閉的 Windows #31](https://github.com/metasequoiaime/MSIME-Windows/issues/31)、[已關閉的 #39](https://github.com/metasequoiaime/MSIME-Windows/issues/39)。

### 同音字太多，能否輸入一個詞，只取其中一個字？

在“设置 → 输入”開啟“以词定字”。選中目標候選後，按 `[` 送出第一個漢字，按 `]` 送出最後一個漢字。例如用容易找到的詞來定位首字或末字。

此功能取的是首尾漢字，不能用它直接選出三字或更長詞語的中間字。

來源：[已關閉的 Windows #14](https://github.com/metasequoiaime/MSIME-Windows/issues/14)、[以詞定字指南](https://msime.app/zh-TW/docs/windows/#以詞定字)。

## 翻譯、資料與回報

### 候選詞的英文釋義不準確，可以自己糾正嗎？

可以新增本地覆蓋檔案：`%LOCALAPPDATA%\metasequoiaime\custom_translations.txt`。使用 UTF-8 文字，每行以真正的 Tab 分隔源詞和釋義；不要把空格或文字“Tab”當成分隔符。

```text
你好	hello
谢谢	thank you
```

相同源詞取最後一條，覆蓋內容優先於內建釋義。儲存後重啟輸入法。#75 記錄了部分高頻詞的覆蓋修正，但不意味著全部翻譯資料都已正確；錯譯仍可附具體詞例回報。

來源：[已關閉的 Windows #75](https://github.com/metasequoiaime/MSIME-Windows/issues/75)、[相關回報 #71](https://github.com/metasequoiaime/MSIME-Windows/issues/71)、[候選詞翻譯指南](https://msime.app/zh-TW/docs/windows/#候選詞翻譯)。

### 快捷片語可以批次匯入嗎？

可以。在“实用功能 → 快捷短语”中使用“批量导入”。匯入 UTF-8 純文字，不加表頭，每行用真正的 Tab 分隔編碼、短語和權重，例如：

```text
mail	example@example.com	10
```

來源格式不同時，先轉換為水杉支援的格式，不能把任意輸入法的詞庫檔案直接當成快捷片語檔案匯入。返回失敗行號時，檢查列數和 Tab，修正後再匯入。

來源：[已關閉的 Windows #29](https://github.com/metasequoiaime/MSIME-Windows/issues/29)、[快捷片語指南](https://msime.app/zh-TW/docs/windows/#快捷片語k-模式)。歷史 Issue 的入口名稱可能與新版不同，以目前指南和安裝版本為準。

### 斷網還能打字嗎？雲端候選字、翻譯或語音失敗怎麼辦？

本地詞庫輸入仍可使用。先關閉出問題的聯網功能，確認普通輸入正常，再核對服務地址、模型、憑證和網路狀態。語音、線上翻譯等依賴配置的服務，不能把服務不可用等同於輸入法整體無法使用。

需要離線輸入時，關閉雲端候選字、AI 聯想、線上翻譯、線上語音辨識及潤色等聯網選項。Windows 指南註明雲端候選字預設開啟，會傳送正在輸入的拼音串。回報網路錯誤時遮擋 Token、SecretKey 等憑證。

依據：[Windows 指南 · 更新、備份與故障排查](https://msime.app/zh-TW/docs/windows/#更新備份與故障排查)、[雲端候選字](https://msime.app/zh-TW/docs/windows/#雲端候選字)。

### 上面的方法都沒解決，回報時需要提供什麼？

準備輸入法版本、作業系統版本、輸入方案、重現步驟、預期結果和實際結果，附能說明現象的截圖。應用相容問題請增加應用名稱、版本、權限狀態；顯示問題增加字型、縮放比例和渲染方式。

先在[Windows Issues](https://github.com/metasequoiaime/MSIME-Windows/issues)搜尋相同問題，再提交 Bug。功能改進可用[官網需求上報](https://msime.app/zh-TW/feedback/)。macOS 和 Linux 請分別檢視[macOS 指南](https://msime.app/zh-TW/docs/macos/)、[Linux 指南](https://msime.app/zh-TW/docs/linux/)。

公開前遮擋截圖中的個人內容、日誌裡的憑證，不要直接附完整使用者詞庫或配置檔案。安全漏洞請按[安全策略](https://github.com/metasequoiaime/.github/blob/main/SECURITY.md)私下報告。

本頁於 2026-09-08 核對 Windows 已關閉 Issue、維護者回復及現行指南，並補充仍開放的字型問題 #232。歷史版本號只說明對應回報的範圍，不代表目前最新版；是否進入你使用的安裝套件，以 Release 為準。
