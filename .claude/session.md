# Session 進度記錄
## 狀態: 已完成
## 最後更新: 2026-06-24 01:16
## 當前分支: main

### 當前任務
- 將 FIFA 2026 賽程表 HTML 部署為線上網站(GitHub Pages),並補齊文件。已完成。

### 已完成
- [x] 分析 `index.html` 中賠率(先驗)與貝葉斯(後驗)的實際實作邏輯
- [x] 撰寫 `docs/fifa2026_odds_bayes_analysis.md`(賠率/貝葉斯分析說明,含與交接文件的差異)
- [x] `git init` + 連結 remote `https://github.com/jokerjkeeper/fifa.git`,本地 main 追蹤 origin/main
- [x] commit docs 文件
- [x] 將 `docs/fifa2026_schedule.html` 移至根目錄 `index.html`(單一來源,作為 Pages 入口)
- [x] 更新 README(線上瀏覽/本地開啟/功能/文件連結)與交接文件檔案清單路徑
- [x] commit 並 push 到 main(經用戶本次授權)
- [x] 用戶手動啟用 GitHub Pages,確認網站運作正常

### 待辦
- [ ] (安全)Odds API Key `023ee379…` 公開暴露於 repo 與線上網站 → 改後端 proxy 或先換新 key
- [ ] (功能)32 強淘汰賽分頁、小組積分表
- [ ] (技術)Odds API 加 localStorage cache(免費額度 500 次/月)
- [ ] (一致性)讓歷史頁也顯示貝葉斯、啟用未使用的 `expected_wins`、補齊爆冷場次的 `ph` 欄位

### 決策記錄
| 日期 | 決策 | 原因 | 替代方案 |
|------|------|------|----------|
| 2026-06-24 | app 移至根目錄 index.html(單一來源) | 避免 docs 與根目錄各存一份 HTML 需各自維護 | 保留 docs 版 + Pages 從 /docs 發佈 |
| 2026-06-24 | GitHub Pages 從 main /(root) 發佈 | 單檔零依賴靜態站,最省事 | gh-pages 分支(本機無 gh CLI) |
| 2026-06-24 | 本次授權 push main | 全域設定禁止 push main,但網站上線需推 main,用戶明確授權 | 用戶自行 push |

### 已知問題
- Odds API Key 明文暴露於前端與公開 repo — 未處理,等待用戶決定方案
- 本機無 `gh` CLI,無法以程式啟用 Pages — 已由用戶手動啟用解決
