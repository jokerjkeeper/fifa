# Session 進度記錄
## 狀態: 進行中（#172 完成，誠實負結果 → 路線收斂到「信賠率」）
## 最後更新: 2026-06-27 20:10
## 當前分支: main

### 當前任務
- 貝葉斯演算法的「資料底座 + 驗證」工程。本次完成：真實歐賠（OddsPortal）匯入 ALL_GAMES，並以 OddsPortal 為權威資料源校正賽程/比分。
- **賠率匯入已完成**：38 場 final 內聯 `odds:{home,draw,away}`，下一步可做回測（#171）。

### 已完成
- [x] 通讀現有貝葉斯實作：`oddsToProb()`(去抽水→先驗) + `bayesUpdate()`/`perfFactor`(啟發式打分→後驗)。結論：非真貝葉斯，係數人工拍定、無法校準
- [x] 對照用戶 `J:\git\obsi\dropbox\項目\數學\隨機\隨機.md`，評估替代方法：Poisson/Dixon-Coles(首選)、Elo/Glicko、Bradley-Terry、ML、Brier 回測
- [x] 新增文檔 `docs/fifa2026_prediction_methods.md`（現況+問題+替代方法+演進路線）
- [x] 歷史頁新增「賽前先驗→貝葉斯」欄：`historyOddsHtml()`，顯示先驗/後驗條+實際結果標示+✓/✗ 命中標記；表頭/colspan(7→8)/i18n 都已補
- [x] `getOddsForGame()` 新增讀取 `g.odds:{home,draw,away}` 欄位（final 場次賽前賠率的權威來源）
- [x] `historyOddsHtml()` 先驗優先序：`g.odds` → 舊 `ph/pd/pa` → 無
- [x] 產出待填清單 `docs/fifa2026_odds_to_fill.csv`（42 場，odds 三欄留空）
- [x] 開立工單 #169~#174（tag fifa2026），#169 進度更新至 40%

### 已完成（本次 2026-06-27 下午）
- [x] 用戶用 OddsPortal 截圖（docs/odds_screenshots/ 3 張，6/12–6/25）提供真實 1X2 賠率，我 OCR 抄出
- [x] 38 場 final 賠率內聯進 ALL_GAMES（`odds:{home,draw,away}`，getOddsForGame line 714 第一優先讀取）
- [x] 以 OddsPortal 為權威源：刪 4 場賽程不符（GER-ECU/CUW-CIV/USA-TUR/AUS-PAR，OddsPortal 無此對戰）
- [x] 修 2 場比分（AUS-TUR 2-1→2-0、GER-CIV 4-1→2-1）
- [x] staging CSV `docs/fifa2026_odds_to_fill.csv` 同步刪 4 場/改 2 比分
- [x] node 語法驗證通過：48 場(38 final 全帶 odds + 10 scheduled)

### 待辦
- [x] #173 修 leakage：buildTeamProfiles(cutoffTs) 只納入開賽前 final；各 html helper 以 kickoffTs(g) 為 cutoff 自建 profiles（不再共用全域）。node 驗證：建檔場數隨時序遞增(0→37)，upcoming 納入全部 38
- [x] #171 Brier/log-loss 回測完成。結果：裸賠率 Brier 0.560 / log 0.919 / 命中 63.2%；貝葉斯 0.578 / 0.929 / 57.9% → **三項全變差**。perfFactor 啟發式無校準價值。做成歷史頁頂常駐卡片 computeBacktest()/renderBacktest()
- [x] #172 Elo→λ→Poisson 模型完成。實作 buildEloRatings/eloToLambda/poissonMatrix/modelProbForGame（walk-forward 無洩漏）；回測卡片擴成三方對比（裸賠率/貝葉斯/Poisson 模型）+ i18n(btCol3/btModelBetter/btModelWorse)。**結果（靜態 38 場）：模型 0.586/0.976/52.6%，即使全樣本選參仍三項全敗，輸給裸賠率 0.560**。根因：每隊僅 3 場 + Elo 1500 冷啟動，盤口已含全球資訊。雙軌交叉驗證一致、瀏覽器目視通過。文檔補第六章
- [x] **（依 #172 結論的收尾）** UI 後驗欄改直接採用裸賠率：oddsInnerHtml/historyOddsHtml 移除 🧮 貝葉斯條，只留 📊 賠率 + 命中標記；表頭/info-box/footer 文案改寫；bayesUpdate/Elo 區塊加 DEPRECATED 註解（函式保留供回測卡片）。三方回測卡片保留為科學記錄。瀏覽器目視通過
- [ ] 6/24–6/25 的 scheduled 場次目前仍用 FALLBACK_ODDS（10 場），未來開賽後可補真實賠率
- [ ] ph/pd/pa 在有 odds 後僅供「爆冷歸因」(line 528-530) 使用；未來換掉啟發式引擎時一併清理
- [ ] （承上 session）Odds API Key 明文暴露、32 強淘汰賽分頁、localStorage cache、隊名正規化
- [ ] 本次程式變更尚未 commit（用戶未要求）

### 決策記錄
| 日期 | 決策 | 原因 | 替代方案 |
|------|------|------|----------|
| 2026-06-27 | 資料底座走「用戶提供真實歷史賠率」 | 最嚴謹、驗證結果可信 | 手動估計先驗(打折)、只記錄未來場次 |
| 2026-06-27 | 存「原始十進位賠率 odds 欄位」而非預算先驗 | 渲染時用 oddsToProb 即時換算，較乾淨、單一真相 | 直接存 ph/pd/pa（舊 8 場做法，保留為 fallback） |
| 2026-06-27 | 歷史頁先驗優先序 odds→ph/pd/pa→無 | 相容舊 8 場估計值，又能吃新真實賠率 | 只認 odds（會讓舊 8 場失效） |
| 2026-06-27 | 以 OddsPortal 為權威資料源，刪 4 場賽程不符＋改 2 場比分 | 既然要當參考數據來源，須與可查證的真實賽果一致，回測才可信 | 保留自有賽程（賠率對不上，回測虛假） |
| 2026-06-27 | 賠率直接 bake 進 ALL_GAMES（內聯 odds 欄位）而非 runtime 匯入腳本 | 靜態網站無持久層，內聯最乾淨、單一真相 | 額外 JS 在載入時 merge（多餘、易出錯） |
| 2026-06-27 | #172 走 Elo→λ→Poisson（非 Dixon-Coles MLE / 非賠率錨定） | 稀疏資料下最穩健；獨立於賠率才能公平驗證有無增量價值 | DC MLE（資料太少估不準）、賠率錨定 λ（等於複製盤口） |
| 2026-06-27 | 模型敗北後採「誠實負結果」收尾，不硬塞贏 | 與 #171 科學精神一致；38 場稀疏資料盤口本就難贏 | 調參硬湊、改用賠率錨定假裝有效 |

### 已知問題
- ~~42 場 final 多數無賠率~~ 已解決：38 場 final 全有真實賠率（OddsPortal）
- 資料源差異：OddsPortal 小組賽配對與原 ALL_GAMES 有出入（已刪 4 場對齊）；若未來要補回那 4 隊的完整小組賽，需用 OddsPortal 真實對戰（GER-CUW/CIV-ECU、USA-AUS/TUR-PAR）重建
- 「自動記錄未來場次 live 賠率快照」靜態網站做不到（無後端持久化）；若要做需改非靜態方案 — 已向用戶說明，暫不影響當前路徑
- buildTeamProfiles 用全賽季建檔有 leakage，回測前須修（#173）
- （承上）Odds API Key 明文暴露；隊名白名單比對易漏接
