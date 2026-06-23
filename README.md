# 🌀 puti-weather-typhoon

讓你的 AI agent 具備颱風追蹤 + 在地天氣報報的能力。

**不限平台** — ChatGPT、Claude、Gemini、OpenCode、任何能上網的 AI 都能用。

## 特色

- **只查 2 個中央氣象署網頁**，快速不亂繞
- **第一次對話自動問你住哪裡**，設定一次就好
- **智慧衝擊分級**：知道颱風離你家多遠、在哪個方向
- **在地天氣**：目前氣溫、降雨機率、未來三天預報
- **人話回報**：不用看經緯度，直接告訴你嚴不嚴重

## 檔案

| 檔案 | 說明 |
|------|------|
| `INSTRUCTIONS.md` | 通用 AI 指令集 — 貼給你的 agent 用（核心） |
| `tid_mapping.json` | 全台 22 縣市 368 鄉鎮 TID 代碼 + 經緯度座標 |
| `config.example.json` | 設定檔範例 |
| `state.example.json` | 追蹤狀態範例 |

## 各平台安裝方式

### 🤖 OpenCode
把 `INSTRUCTIONS.md` 的內容貼到 custom instructions 或做成 skill。
把 `tid_mapping.json` 放在 agent 能讀到的路徑。

### 🤖 ChatGPT (Custom GPT)
在 GPT 的 Instructions 欄位貼上 `INSTRUCTIONS.md` 全文。
在 Knowledge 區上傳 `tid_mapping.json`。
GPT 有網頁瀏覽功能，會自動去 CWA 抓資料。

### 🤖 Claude (Projects)
在 Project 的 Custom Instructions 貼上 `INSTRUCTIONS.md` 全文。
上傳 `tid_mapping.json` 作為知識庫檔案。

### 🤖 Gemini (Gems)
在 Gem 的 Instructions 貼上 `INSTRUCTIONS.md` 全文。
Gemini 可啟用 Google 搜尋或自訂工具來抓取網頁資料。

### 🤖 自行開發
直接叫你的 AI 讀 `INSTRUCTIONS.md` 的指示，用 `tid_mapping.json` 作為參考資料，
實作網頁抓取邏輯去 CWA 網站取得天氣資料。

## 授權

MIT — 自由使用、修改、分享。
