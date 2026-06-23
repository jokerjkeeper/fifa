# FIFA 世界盃 2026 · 賽程表

純前端單頁應用,追蹤 FIFA 2026 世界盃賽程、即時比分、外盤賠率,並透過貝葉斯推斷動態修正勝率預測。

## 線上瀏覽

GitHub Pages:`https://jokerjkeeper.github.io/fifa/`

## 本地開啟

直接用瀏覽器打開 `index.html` 即可,零依賴、無需建置。

## 功能

- **三分頁賽程**:即將開賽(組別篩選)/ 今日賽事(自動跳轉)/ 歷史紀錄(爆冷·大比分·平局篩選)
- **外盤賠率**:載入時自動從 The Odds API 抓歐洲盤 h2h 賠率,失敗則 fallback 內建備用賠率
- **貝葉斯修正**:以賠率隱含概率為先驗,疊加本屆各隊真實戰績修正勝率(🔵 先驗 vs 🟣 後驗)

## 文件

- [`docs/fifa2026_dev_handoff.md`](docs/fifa2026_dev_handoff.md) — 開發交接文件
- [`docs/fifa2026_odds_bayes_analysis.md`](docs/fifa2026_odds_bayes_analysis.md) — 賠率與貝葉斯分析說明

## 技術棧

純 HTML / CSS / JS,單一檔案 `index.html`,零依賴、無打包流程。
