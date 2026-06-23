# FIFA 世界盃 2026 賽程表 — 開發交接文件

## 專案概覽

一個純前端單頁 HTML 應用，追蹤 FIFA 2026 世界盃賽程、即時比分、外盤賠率，並透過貝葉斯推斷動態修正勝率預測。

---

## 已完成功能（第一、二階段）

### 第一階段：賽程表

**三分頁結構**
- **即將開賽**：未來所有待踢比賽，含組別篩選（A–L）
- **今日賽事**：依當天日期自動篩選，打開時若有今日賽事自動跳轉此頁
- **歷史紀錄**：所有已結束比賽（6/11 起），最新排最前，可篩選「爆冷」「大比分（3+球）」「平局」

**資料結構（每場比賽物件）**
```js
{
  s: 'final' | 'scheduled' | 'live',
  t: '2026-06-24T01:00',   // 台灣時間，格式 YYYY-MM-DDTHH:mm
  h: 'POR',                 // 主隊縮寫
  a: 'UZB',                 // 客隊縮寫
  hs: 0, as: 0,             // 比分
  upset: true,              // 選填，爆冷標記
  ph: 85.8, pd: 10.0, pa: 4.2  // 選填，先驗勝率（歷史場次用）
}
```

**爆冷自動標記邏輯**
```js
const UPSETS = new Set(['ESP-CPV','ECU-CUW','BEL-IRN','URU-CPV','AUS-PAR']);
// 格式：'主隊-客隊'，手動維護
```

**統計列**：總場次 / 已結束 / 直播中 / 即將開賽，即時計算顯示

---

### 第二階段：外盤賠率 + 貝葉斯修正

#### 賠率來源：The Odds API
```
API Key: 023ee379fbdbea2274371b447449b272
Endpoint: https://api.the-odds-api.com/v4/sports/soccer_fifa_world_cup/odds/
Params: regions=eu&markets=h2h&oddsFormat=decimal
```
- 頁面載入時自動 fetch，失敗則 fallback 到內建備用賠率
- 狀態顯示於 header（載入中 / 已載入N場 / 離線備用）

#### 賠率 → 概率換算（去除莊家抽水）
```js
function oddsToProb(home, draw, away) {
  const raw_h = 1/home, raw_d = 1/draw, raw_a = 1/away;
  const total = raw_h + raw_d + raw_a;
  return {
    ph: raw_h/total*100,
    pd: raw_d/total*100,
    pa: raw_a/total*100
  };
}
```

#### 貝葉斯更新引擎

**先驗（Prior）** = 賠率隱含概率

**似然函數（Likelihood）** = 依本屆已完成比賽建立每隊表現檔案：
```js
// 每隊 profile：games, wins, draws, losses,
//               goals_for, goals_against,
//               upsets_caused, upset_suffered

function perfFactor(team, outcome) {
  // outcome: 'win' | 'draw' | 'loss'
  // 勝率、進失球差、爆冷次數 → 綜合係數
  // 回傳值 > 1 表示該結果比預期更可能發生
}
```

**後驗（Posterior）** = 正規化後的貝葉斯更新：
```js
unnorm_h = P(H勝|prior) × L(H勝|H隊表現) × L(A敗|A隊表現)
unnorm_d = P(平|prior) × L(平|H隊) × L(平|A隊)
unnorm_a = P(A勝|prior) × L(A勝|A隊) × L(H敗|H隊)
posterior = normalize(unnorm_h, unnorm_d, unnorm_a)
```

**修正顯示**
- 🔵 藍色條 = 賠率先驗概率
- 🟣 紫色條 = 貝葉斯後驗概率
- ↑↓ 箭頭 = 差距 > 1.5% 才顯示

---

## 視覺主題

- **配色**：淡色主題，背景 `#f0f4fb`，卡片白底
- **字型**：系統字體堆疊（-apple-system, Segoe UI, PingFang TC）
- **框架**：純 HTML/CSS/JS，零依賴，無 npm/打包流程

---

## 資料維護說明

### 新增比賽結果
在 `ALL_GAMES` 陣列中將 `s:'scheduled'` 改為 `s:'final'`，填入 `hs`/`as` 比分：
```js
{s:'final', t:'2026-06-24T01:00', h:'POR', a:'UZB', hs:3, as:0}
```

### 新增爆冷標記
```js
{s:'final', ..., upset:true, ph:85, pd:10, pa:5}
// ph/pd/pa = 當時的先驗勝率，供貝葉斯回溯計算使用
```

### 新增即將開賽場次
```js
{s:'scheduled', t:'2026-06-27T03:00', h:'ARG', a:'JOR', hs:0, as:0}
// 賠率會自動從 Odds API 拉取，無需手動填入
```

---

## 目前資料覆蓋範圍

| 輪次 | 日期 | 場次 | 狀態 |
|------|------|------|------|
| 第一輪 | 6/11–6/18 | 24場 | ✅ 已完成 |
| 第二輪 | 6/18–6/23 | 16場 | ✅ 已完成 |
| 第三輪 | 6/24–6/25 | 10場 | 🔵 即將開賽 |
| 32強淘汰賽 | 6/28起 | 16場 | ⬜ 未加入 |

---

## 後續開發建議

### 功能面
1. **32強/16強淘汰賽分頁** — 新增第四個分頁「淘汰賽」，顯示對陣圖
2. **小組積分表** — 依現有比賽結果自動計算各組積分排名
3. **貝葉斯參數調整介面** — 讓使用者手動調整 perfFactor 的權重係數
4. **比賽結果快速輸入** — 在頁面內直接輸入比分，自動更新 `ALL_GAMES` 狀態（可用 localStorage 儲存）
5. **Odds API 自動輪詢** — 每5分鐘自動重新拉取賠率，直播中比賽即時更新

### 技術面
1. **API Key 安全** — 目前 Key 直接暴露在前端，正式部署建議透過後端 proxy 轉發
2. **Odds API 免費額度** — 500次/月，建議加入 cache（localStorage + timestamp），避免每次重開頁面都消耗配額
3. **SportRadar 整合** — 目前比分為手動維護，可接 SportRadar API 自動拉取即時比分（需另取 API Key）

---

## 檔案清單

```
index.html                       ← 主檔案，單一 HTML 含所有 CSS/JS（網站入口）
docs/fifa2026_dev_handoff.md     ← 本文件
docs/fifa2026_odds_bayes_analysis.md ← 賠率與貝葉斯分析說明
```
