---
name: typhoon-tracker
description: 颱風追蹤技能。安裝後第一次對話會詢問你的居住地，之後監控颱風動態並根據你的所在地回報衝擊程度與天氣影響。
---

# 🌀 颱風追蹤技能 (typhoon-tracker)

一個開源的 OpenCode 技能，讓你每次跟 AI 聊天時自動附帶颱風快報。

**只查中央氣象署 2 個網頁**，不用 Wikipedia 或其他來源。

---

## 📦 安裝方式

1. 把整個 `typhoon-tracker/` 資料夾複製到你的 OpenCode skills 路徑
2. 在 opencode 設定中加入技能路徑
3. 重啟 opencode
4. 跟 AI 說第一句話，它就會問你：「你家在哪個縣市、哪個鄉鎮區？」

---

## ⚙️ 設定機制（第一次對話自動觸發）

Agent 第一次載入技能時，偵測到沒有 `config.json`，會主動問使用者：

> 「颱風追蹤技能需要設定你的居住地，才能提供在地化的天氣與衝擊評估。請問你住在哪個縣市、哪個鄉鎮區呢？」
>
> 例如：臺北市大安區、屏東縣里港鄉、高雄市左營區

使用者回答後，Agent 自動查 `data/tid_mapping.json` 找出對應的 TID 與座標，寫入 `config.json`。

**支援多個地點**（例如老家 + 現在住處），可後續用「新增地點」「更新地點」管理。

如果使用者輸入的地名不在對照表中，Agent 引導使用者：
> 「請到中央氣象署鄉鎮預報網頁 https://www.cwa.gov.tw/V8/C/W/Town/Town.html 選取你的鄉鎮，然後把網址列 ?TID=XXXXXXX 的數字告訴我。」

---

## 📁 檔案結構

```
typhoon-tracker/
├── SKILL.md              ← 本技能檔案
├── config.json           ← 使用者設定（安裝後自動產生）
├── config.example.json   ← 設定範例
├── state.json            ← 追蹤狀態（啟用/停止）
├── data/
│   └── tid_mapping.json  ← 全台 22 縣市 368 鄉鎮 TID 對照表 + 座標
└── README.md             ← 安裝與使用說明
```

---

## 🌐 資料來源

**只查以下 2 個中央氣象署網頁：**

| 用途 | 網址 |
|------|------|
| 颱風消息 | `https://www.cwa.gov.tw/V8/C/P/Typhoon/TY_NEWS.html` |
| 鄉鎮天氣 | `https://www.cwa.gov.tw/V8/C/W/Town/Town.html?TID={你的TID}` |

⚠️ 兩個網頁都是 JavaScript 動態渲染，**不能用 webfetch**。
必須用 **Playwright 瀏覽器工具**：`playwright_browser_navigate` → `playwright_browser_evaluate` 取 `body.innerText`。

---

## 📊 資料解析方式

### 颱風資料
從 `body.innerText` 找「目前 (民國...) 太平洋地區有 X 個颱風」段落。
每個颱風的資料：
```
中度颱風米克拉 編號第 07 號 國際命名 MEKKHALA
→ 24日2時的中心位置在北緯 20.2 度，東經 124.6 度，
  以每小時9公里速度，向北進行。中心氣壓945百帕，
  近中心最大風速每秒43公尺...
```

### 鄉鎮天氣資料
從 `body.innerText` 找：
- **即時觀測**：溫度、體感溫度、相對溼度、時雨量（表格上方區塊）
- **天氣概況按鈕**：例如「多雲午後短暫雷陣雨，天氣整體舒適，但仍有降雨機率」
- **降雨機率列**：從「逐三小時天氣預報資料」表格找「降雨機率」那一行
- **溫度列**：從表格找「溫度」那一行，取今日最高最低
- **三日資料**：表格中 Day1=今天, Day2=明天, Day3=後天

---

## 📐 距離計算與方向判斷

使用 `data/tid_mapping.json` 中各縣市的中心座標。

```javascript
function distanceFromUser(typhoonLat, typhoonLng, userLat, userLng) {
  const R = 6371; // 地球半徑 km
  const dLat = (typhoonLat - userLat) * Math.PI / 180;
  const dLng = (typhoonLng - userLng) * Math.PI / 180;
  const a = Math.sin(dLat/2)**2 + Math.cos(userLat*Math.PI/180) * Math.cos(typhoonLat*Math.PI/180) * Math.sin(dLng/2)**2;
  return R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
}

function bearing(typhoonLat, typhoonLng, userLat, userLng) {
  const dLng = (typhoonLng - userLng) * Math.PI / 180;
  const y = Math.sin(dLng) * Math.cos(typhoonLat * Math.PI / 180);
  const x = Math.cos(userLat * Math.PI / 180) * Math.sin(typhoonLat * Math.PI / 180) -
            Math.sin(userLat * Math.PI / 180) * Math.cos(typhoonLat * Math.PI / 180) * Math.cos(dLng);
  const brng = Math.atan2(y, x) * 180 / Math.PI;
  const dirs = ['北', '東北', '東', '東南', '南', '西南', '西', '西北'];
  return dirs[Math.round(((brng + 360) % 360) / 45) % 8];
}
```

---

## 🚦 衝擊分級

根據颱風與使用者所在地的距離自動分級：

| 分級 | 距離 | 圖示 | 說明 |
|------|------|------|------|
| 單純關注 | >800km | 🟢 | 沒感覺，當氣象新聞看 |
| 注意 | 500~800km | 🟡 | 外圍環流可能開始影響 |
| 警戒 | 300~500km | 🟠 | 暴風圈邊緣接近中 |
| 威脅 | <300km | 🔴 | 暴風圈可能觸及 |
| 直擊 | 路徑經過 | ⚫ | 暴風圈直接 hit |

**方向描述**：根據 `bearing()` 計算颱風在你家的哪個方向。
例如「颱風在你家東南方約 400 公里」

---

## 🗣️ 回報格式

**人話優先，不要丟數字。**

```
🌀 米克拉快報
⚠️ 影響分級：🟠 警戒（距你家約 400 公里）
📍 在你家東南方海面上，向北龜速移動（9km/h，比腳踏車慢）
💨 中颱，風速 43m/s——瘦子出門會被吹倒的程度
🌪️ 暴風圈半徑 180 公里

🌤 你所在地現在
🌡️ 28°C（體感 34°C，悶熱）
💧 降雨機率：早上 10% → 午後 60%（下午 4 點前後高峰）
📅 未來三天：今天 27~34°C 午後雷陣雨
              明天 26~32°C 短暫陣雨
              後天 25~32°C 多雲午後陣雨
```

---

## 🎯 啟用與停止

| 指令 | 效果 |
|------|------|
| `注意[颱風名]` / `追蹤[颱風名]` | 開始追蹤該颱風 |
| `停` / `取消追蹤` / `不用看了` | 停止追蹤 |
| `新增地點` / `更新地點` | 重新設定居住地 |

**自動回報**：每次使用者對話時，先檢查 `state.json`，有 active 追蹤就附帶快報。

**自動過期**：連續 3 次查不到該颱風資料，自動設為 expired 並告知。

---

## 🤝 貢獻

歡迎 PR！可以貢獻的方向：
- 補充各鄉鎮的精確座標（目前用縣市中心，可精細到鄉鎮）
- 支援其他語系（英文版）
- 整合颱風路徑預測圖
