# 路邊野貓 AI（AlleyCat）— DaVinci Resolve 字幕管理外掛

> 代號 **AlleyCat**,檔名/選單名仍為 **SubFix**。一支以 Lua 撰寫、執行於 DaVinci Resolve 20 Fusion 腳本環境的字幕管理工具。
> 本倉庫提供**繁體中文(台灣用語)**版本,介面文字、註解、日誌與 AI 提示詞皆已在地化,並完成品牌改名(原名 Hooper AI 2.0)。

---

## 這是什麼

SubFix 是一支單檔 Lua 腳本(約 12,300 行、284 個函式),安裝到 DaVinci Resolve 後,會在 `Workspace → Scripts` 選單中出現,開啟後是一個原生視窗面板,讓你在不離開達文西的情況下,集中管理時間軸上的字幕:批次搜尋、定位跳轉、整段取代,並可呼叫大型語言模型(LLM)做**全自動字幕糾錯**。

原作者以「vibe coding」(主要由 AI 協助撰寫程式)的方式,花約兩個月完成此外掛並免費分享。本倉庫在其基礎上完成繁體中文與台灣用語在地化。

---

## 它有用到 AI 嗎?

**有,而且是兩層意義上的:**

1. **開發層面** — 原作者是借助 AI 寫出這支外掛的。
2. **執行層面(重點)** — 外掛內建一套「AI 糾錯引擎」。當你按下「開始 AI 處理」,它會把字幕內容送到你指定的大模型 API(DeepSeek / OpenAI / Gemini 等)進行糾錯、修正錯別字與標點。**要使用此功能,你必須自備一組 API 金鑰。**

不需要 AI 的功能(載入字幕、搜尋、定位、批次取代、備份還原、匯出 SRT)則全在本機完成,不會連網。

---

## 主要功能

| 引擎 | 功能 |
|------|------|
| **UI 引擎** | 以 Fusion 內建 `UIManager` + `UIDispatcher` 繪製原生視窗(TabBar + Stack 分頁架構) |
| **時間碼引擎** | 影格 ↔ SRT/SMPTE 時間碼互轉、支援多種影格率與 Drop-frame、把修改寫回時間軸 |
| **時光機備份引擎** | 每次 AI 處理前自動備份字幕,可從歷史紀錄一鍵還原、撤回或清空 |
| **AI 糾錯引擎** | 透過背景 `curl` 子程序呼叫 LLM,支援多家供應商與自訂端點,可隨時中止 |

附加能力:即時搜尋過濾、雙擊定位跳轉、批次取代、中英排版間距最佳化、清理空字幕塊、匯出 SRT、純文字複製到剪貼簿、中↔英 AI 翻譯、**口頭禪一鍵清理(本版新增)**。

### 本版新增 / 強化

**口頭禪一鍵清理**:精修工具面板新增「口頭禪清理」按鈕,逐行移除語助詞與口吃疊詞(嗯、呃、那個那個、就是就是…),會列出修改報告供複核,並可一鍵撤回。要增刪詞表,直接改 `SubFix.lua` 裡 `win.On.BtnFillerClean.Clicked` 開頭的 `FILLER_WORDS` / `FILLER_PHRASES` 兩個表即可。

**中英雙語=兩條獨立字幕軌(用既有機制達成,免新程式)**:插件本來就有「翻譯:中→英」任務,以及「更新到軌 N」目標軌選擇(會自動建軌)。要做到 ST1 中文、ST2 英文兩條軌,流程是:

1. 按「重新整理字幕」載入原文(在字幕軌 1)。
2. AI 工作台選「3. 翻譯:中 → 英」→ 開始 AI 處理(記憶體中的文字變成英文)。
3. 把「更新到軌」設成 **2**。
4. 按「更新時間線」→ 英文落到軌道 2,原文中文留在軌道 1。

> 注意:達文西同一時間只會顯示一條字幕軌,所以兩條軌是給你「切換語言」或「分別輸出中/英版」用的,不是同畫面雙語。要同畫面雙語請改用「同一字幕兩行」或「字幕區域(Subtitle Region)」。

---

## 系統需求

- **macOS**(原作者僅提供 Mac 測試環境;Windows 未適配)
- **DaVinci Resolve 20.x**(使用其內建 Fusion / Lua 腳本環境,腳本為 Lua 5.2+)
- 系統內建 `curl`(macOS 預設即有)
- 若要用 AI 功能:一組可用的 LLM API 金鑰

---

## 安裝

### 方式一:手動放置腳本(推薦,適合自管版本)

將 `SubFix.lua` 複製到下列任一資料夾:

```
~/Library/Application Support/Blackmagic Design/DaVinci Resolve/Fusion/Scripts/Utility/
```
或
```
~/Library/Application Support/Blackmagic Design/DaVinci Resolve/Fusion/Scripts/Comp/
```

重新啟動 DaVinci Resolve 即可。

### 方式二:原版 .pkg 安裝包

原作者也提供 `.pkg` 安裝包(識別碼 `com.mediastorm.subfix`),會自動把腳本安裝到上述 `Utility/` 路徑,並附帶一支 `Uninstall_SubFix.app` 用於移除。

---

## 啟動

在 DaVinci Resolve 中開啟:

```
Workspace(工作區）→ Scripts → SubFix
```

面板開啟後,先按「重新整理字幕」載入當前時間軸的字幕軌。

---

## 設定 AI(選用)

AI 功能(完整糾錯、的得專項、中英翻譯)需要自備一組 LLM API 金鑰。**這是「按用量計費」的 API 金鑰,跟 Claude Pro / ChatGPT Plus 那種「月費訂閱」是兩回事——訂閱不能拿來接 API。** 字幕糾錯用量很小,一支影片大約幾美分,儲值制反而比月費划算。

在面板「AI 工作台 → 配置」填入引擎、模型、Key 即可(會自動儲存)。內建引擎:

| 引擎 | 建議模型 | 協定 | 取得金鑰 |
|--------|----------|------|----------|
| DeepSeek | `deepseek-v4-pro`(好)/ `deepseek-v4-flash`(省) | OpenAI 相容 | <https://platform.deepseek.com/api-keys> |
| Google Gemini | `gemini-2.5-flash`(免費級)/ `gemini-2.5-pro`(更好) | Gemini 原生 | <https://aistudio.google.com/app/apikey> |
| OpenAI | `gpt-4o-mini` 等 | OpenAI 相容 | <https://platform.openai.com/api-keys> |
| SiliconFlow | 填模型廣場複製的 ID,如 `deepseek-ai/DeepSeek-V3.2` | OpenAI 相容 | <https://cloud.siliconflow.cn/me/account/ak> |
| 自訂(OpenAI 相容) | 自填 | OpenAI 相容 | 任何相容端點(含 Claude) |

> ⚠️ DeepSeek 官方的舊模型名 `deepseek-chat` / `deepseek-reasoner` 預計 **2026/07/24 停用**,請改填 `deepseek-v4-flash` / `deepseek-v4-pro`。

**各家怎麼接(重點):**

- **Gemini(若你別處已有金鑰,可直接重用)**:引擎選 Google Gemini、模型 `gemini-2.5-flash`、Key 貼 Google AI Studio 的金鑰。免費級約每天 1,500 次、每分鐘 15 次,個人字幕用通常免費就夠;要更好改 `gemini-2.5-pro`(需在 Google 專案開啟計費)。
- **Claude**:用「自訂(OpenAI 相容)」引擎,API 填 `https://api.anthropic.com/v1/chat/completions`、模型 `claude-sonnet-4-6`(或 `claude-haiku-4-5` 省錢)、Key 為 console.anthropic.com 的 API 金鑰。
- **SiliconFlow 換模型**:只改「模型」欄即可,API 與 Key 不動;模型 ID 是「廠商/模型名」格式,去它模型廣場用「複製模型名稱」貼上最準。
- **GLM-5.2(智譜 / Z.ai)**:可經 SiliconFlow 引擎填模型 ID(如 `zai-org/GLM-5.2`),或用「自訂(OpenAI 相容)」填模型 `glm-5.2` + 端點 `https://open.bigmodel.cn/api/paas/v4/chat/completions`、Key 用 bigmodel.cn / z.ai 申請。新用戶 bigmodel.cn 約有 2,000 萬 token 免費額度。價格約 $1.40 / $4.40(每百萬輸入/輸出)。⚠️ GLM 同為中國模型,政治敏感內容的審查與資料疑慮與 DeepSeek 相同,不適合作為「避審查」用途。

**選哪個模型 / 敏感內容注意(台灣創作者特別看):**

- 日常、非政治內容:DeepSeek 最便宜好用,「台灣」一詞本身**不會**被擋。
- **政治/敏感題材(台灣主權、兩岸、六四、批評中共等):建議改用 Gemini 或 Claude。** DeepSeek、GLM 等中國模型會依中國法規對這類「主題」做審查——可能拒答(該行會自動保留原文),或更麻煩地把用詞改成北京立場。而且送往 `api.deepseek.com` 的內容會傳到中國伺服器。Gemini / Claude 無此政治過濾、資料也不進中國。
- 換模型只需改「AI 配置」一格,可日常用 DeepSeek、敏感題材臨時切 Gemini/Claude。

**金鑰儲存位置(僅存於你本機):**

```
~/.alleycat_config.json          # 主設定(JSON)
~/.alleycat_config.txt           # 舊版設定(相容)
~/.alleycat_config_recommended.txt
~/.alleycat_config_custom.txt
```

> 安全性:API 金鑰只寫入你家目錄的設定檔,不會上傳到第三方。AI 請求透過背景 `curl` 送出,執行中可隨時按中止鍵,腳本會 kill 掉該 curl 子程序立即停止。

> 在地化:糾錯與翻譯的提示詞已要求 AI「一律輸出繁體中文(台灣用語、台灣標點)」;「的/地/得」採台灣寬鬆版——副詞用「的」(慢慢的、好好的)會保留、不會被硬改成「地」。

---

## AI 處理流程(八步流水線)

按下「開始 AI 處理」後,腳本依序執行:

1. 讀取與正規化字幕內容
2. 升級 JSON 跳脫與解析、組裝請求
3. 呼叫 LLM 取得糾錯結果
4. 中英文排版間距最佳化
5. 比對差異、產生修改報告
6. 寫回時間軸並建立時光機備份
7. 清理空字幕塊
8. 將純文字結果複製到剪貼簿

每次處理都會產生可回顧的「修改報告」,可逐條檢視、取消部分修改或整批還原。

---

## 倉庫結構建議

```
.
├── README.md
├── SubFix.lua            # 繁體中文(台灣)版主腳本
└── (選用) installer/
    ├── SubFix_v2.3-tw_AlleyCat.pkg
    └── Uninstall_SubFix.app
```

---

## 關於本繁體版

- 以 OpenCC `s2twp`(簡轉繁 + 台灣慣用詞)轉換,並人工校對影視/技術用語:
  視頻→**影片**、軟件→**軟體**、默認→**預設**、刷新→**重新整理**、緩存→**快取**、導出→**匯出**、達芬奇→**達文西** 等。
- 轉換僅變更中日韓文字,**未更動任何程式碼識別字、字串引號或語法結構**(經非漢字骨架逐字元比對確認一致,行數維持 12,327 行),不影響執行。
- 目標執行環境為 DaVinci Resolve 內建的 Lua(腳本使用 `goto`/標籤,屬 **Lua 5.2+**;已用 Lua 5.4 `luac -p` 通過整檔語法檢查)。

---

## 授權與致謝

本外掛由原作者(Bilibili UP 主「小壕h」,內部識別碼 `com.mediastorm.subfix`)製作並**免費分享**。本倉庫為其繁體中文在地化版本,著作權仍屬原作者所有。

- 原始發佈影片:〈我花了兩個月讓 AI 做了個達芬奇外掛?|免費分享〉
- 若你要公開散布或二次發佈,請尊重原作者的分享條款並標註出處。

> 免責聲明:本工具會修改你 DaVinci Resolve 專案中的字幕。請務必先備份重要專案。對任何資料遺失,使用者需自行承擔風險。
