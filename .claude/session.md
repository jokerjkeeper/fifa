# Session 進度記錄
## 狀態: 已完成
## 最後更新: 2026-06-25 22:39
## 當前分支: main

### 當前任務
- 修正「今日賽事」比分刷新與顯示問題。已完成並 push 到 main(commit d28f03d)。

### 已完成
- [x] 診斷波黑(BIH) vs 卡塔爾(QAT)比分刷不出的原因 — 實測 scores API 回傳隊名為 `Bosnia & Herzegovina`(用 `&`),但 `ODDS_NAME_MAP` 只有 `Bosnia and Herzegovina`(用 `and`),名稱對不上 → fetchScores 跳過該場
- [x] 修正:ODDS_NAME_MAP 補上 `'Bosnia & Herzegovina':'BIH'` 與連字號版 `'Bosnia-Herzegovina':'BIH'`(防呆)。實測該場真實比分為 BIH 3:1 QAT(completed)
- [x] 重構 `oddsCellHtml` → 抽出 `oddsInnerHtml(g, profiles)`(不含 `<td>` 外框)共用;新增 `scoreOddsCellHtml`(上比分、下完整賠率+貝葉斯)
- [x] `renderToday`:已開賽(final/live)場次改用 `scoreOddsCellHtml`,保留賠率/貝葉斯供對照(原本只顯示比分)
- [x] commit + push main(用戶本次明確授權),commit d28f03d

### 待辦
- [ ] (安全)Odds API Key `023ee379…` 公開暴露於 repo 與線上網站 → 改後端 proxy 或先換新 key
- [ ] (功能)32 強淘汰賽分頁、小組積分表
- [ ] (技術)Odds API 加 localStorage cache(免費額度 500 次/月)
- [ ] (一致性)讓歷史頁也顯示貝葉斯、啟用未使用的 `expected_wins`、補齊爆冷場次的 `ph` 欄位
- [ ] (健壯性)考慮對 Odds API 隊名做正規化(去除 `&`/`and`/重音差異)以避免類似名稱對映漏接

### 決策記錄
| 日期 | 決策 | 原因 | 替代方案 |
|------|------|------|----------|
| 2026-06-24 | app 移至根目錄 index.html(單一來源) | 避免 docs 與根目錄各存一份 HTML 需各自維護 | 保留 docs 版 + Pages 從 /docs 發佈 |
| 2026-06-24 | GitHub Pages 從 main /(root) 發佈 | 單檔零依賴靜態站,最省事 | gh-pages 分支(本機無 gh CLI) |
| 2026-06-25 | 名稱以「補別名」而非「正規化」方式修 | 改動最小、風險最低,立即解決當前漏接 | 做隊名正規化函式(列入待辦) |
| 2026-06-25 | 已開賽場次保留賠率+貝葉斯(不再只顯示比分) | 用戶要留著賽前盤口與貝葉斯供賽後對照 | 維持原本只顯示比分 |

### 已知問題
- Odds API Key 明文暴露於前端與公開 repo — 未處理,等待用戶決定方案
- 隊名對映採白名單比對,API 改用其他寫法(如 `&`/`and`/重音/縮寫)時會漏接 — 本次已補 BIH,其餘待正規化處理
