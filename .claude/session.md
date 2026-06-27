# Session 進度記錄
## 狀態: 進行中（等待用戶填入歷史賠率）
## 最後更新: 2026-06-27 09:56
## 當前分支: main

### 當前任務
- 貝葉斯演算法的「資料底座 + 驗證」工程。本次完成：(1) 方法論文檔，(2) 歷史頁顯示賠率→貝葉斯，(3) 開立 6 張工單，(4) 搭好「真實賠率」匯入底座並產出待填清單。
- **阻塞點**：等用戶在公司電腦填好 `docs/fifa2026_odds_to_fill.csv`（42 場 final 的真實歐賠）後回傳，我再寫匯入腳本合併進 ALL_GAMES。

### 已完成
- [x] 通讀現有貝葉斯實作：`oddsToProb()`(去抽水→先驗) + `bayesUpdate()`/`perfFactor`(啟發式打分→後驗)。結論：非真貝葉斯，係數人工拍定、無法校準
- [x] 對照用戶 `J:\git\obsi\dropbox\項目\數學\隨機\隨機.md`，評估替代方法：Poisson/Dixon-Coles(首選)、Elo/Glicko、Bradley-Terry、ML、Brier 回測
- [x] 新增文檔 `docs/fifa2026_prediction_methods.md`（現況+問題+替代方法+演進路線）
- [x] 歷史頁新增「賽前先驗→貝葉斯」欄：`historyOddsHtml()`，顯示先驗/後驗條+實際結果標示+✓/✗ 命中標記；表頭/colspan(7→8)/i18n 都已補
- [x] `getOddsForGame()` 新增讀取 `g.odds:{home,draw,away}` 欄位（final 場次賽前賠率的權威來源）
- [x] `historyOddsHtml()` 先驗優先序：`g.odds` → 舊 `ph/pd/pa` → 無
- [x] 產出待填清單 `docs/fifa2026_odds_to_fill.csv`（42 場，odds 三欄留空）
- [x] 開立工單 #169~#174（tag fifa2026），#169 進度更新至 40%

### 待辦
- [ ] **用戶填 `docs/fifa2026_odds_to_fill.csv`**（來源建議：OddsPortal / Betexplorer 的收盤或平均 1X2 賠率）
- [ ] 收到後寫匯入腳本：依 `t` 時間戳把 odds 合併進 index.html 的 ALL_GAMES（工單 #169）
- [ ] #171 Brier/log-loss 回測（裸賠率 vs 貝葉斯）— 資料到齊後做
- [ ] #173 修 leakage（buildTeamProfiles 只用賽前場次）— 回測前必修，否則數字虛高
- [ ] #172 用 Poisson/Dixon-Coles 取代 perfFactor
- [ ] （承上 session）Odds API Key 明文暴露、32 強淘汰賽分頁、localStorage cache、隊名正規化
- [ ] 本次程式變更尚未 commit（用戶未要求）

### 決策記錄
| 日期 | 決策 | 原因 | 替代方案 |
|------|------|------|----------|
| 2026-06-27 | 資料底座走「用戶提供真實歷史賠率」 | 最嚴謹、驗證結果可信 | 手動估計先驗(打折)、只記錄未來場次 |
| 2026-06-27 | 存「原始十進位賠率 odds 欄位」而非預算先驗 | 渲染時用 oddsToProb 即時換算，較乾淨、單一真相 | 直接存 ph/pd/pa（舊 8 場做法，保留為 fallback） |
| 2026-06-27 | 歷史頁先驗優先序 odds→ph/pd/pa→無 | 相容舊 8 場估計值，又能吃新真實賠率 | 只認 odds（會讓舊 8 場失效） |

### 已知問題
- 42 場 final 目前 35 場無賠率→歷史頁顯示「賠率未記錄」；待 CSV 填回後消除（工單 #169）
- 「自動記錄未來場次 live 賠率快照」靜態網站做不到（無後端持久化）；若要做需改非靜態方案 — 已向用戶說明，暫不影響當前路徑
- buildTeamProfiles 用全賽季建檔有 leakage，回測前須修（#173）
- （承上）Odds API Key 明文暴露；隊名白名單比對易漏接
