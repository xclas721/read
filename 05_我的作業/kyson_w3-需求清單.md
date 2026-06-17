# W3 課前作業 — 需求詳細清單（Kyson）

> 綜整 A／B／C 三組衝突裁決 + 全部 11 份訪談紀錄，收斂成結構化需求清單，作為 W3 撰寫 spec 的原料。
> 三段對齊 spec：功能需求 / 非功能需求 / 開放問題。
> 視角：SRE（Kyson），但涵蓋主管 / Owner / SRE 三方。

---

## 基本資料

- 姓名 / 組別：Kyson / SRE 團隊
- 工具：**客戶資訊工具**（AI Agent 生態的基礎識別層，第一階段優先開發）
- 一句目的：AI Agent 收到各管道（email / WeChat / Teams…）來的客訴或告警時，先解析「是哪家客戶、用哪個服務（ACS/3DSS）、找哪個窗口」，回傳結構化身分與行動建議，讓 Agent 判斷後續調用哪些工具——其他工具不知道客戶是誰就沒有意義。

### 範疇邊界（先講清楚，避免 AI 把別工具的事做進來）

- ✅ 本工具做：客戶身分識別、客戶/聯絡人/服務/SLA 資料查詢與維護、AI Summary、CLI/API 介接。
- ❌ **不在本工具**：歷史問題分析（另一個 Developer Site / 工具）、對外寄信（郵件處理工具）、API QA。

---

## 1. 功能需求

> 一條一列。涵蓋「沒人吵但一定要有的 baseline」+「三組衝突取捨後的結論」。寫需求(what)，不寫解法(how)。

| ID | 需求描述 | MoSCoW | 來源 |
|---|---|---|---|
| **客戶識別** | | | |
| F1 | 反查單筆客戶 email／基本資料（最高頻） | Must | baseline；Owner「查 email 最常用、優先」 |
| F2 | 依 email domain 辨識客戶；通訊軟體依群組名稱辨識 | Must | 主管／Jesse |
| F3 | 自動辨識客戶使用的服務（ACS／3DSS／veriid）並做服務配對 | Must | 主管／Andrew |
| F4 | 查無／不在名單 → 在 AI Summary 顯示「查無結果」+ 提示（檢查舊資料／建立新資料／手動輸入），轉人工 | Must | baseline；Alan／Ryan／Wayne |
| **資料模型與維護** | | | |
| F5 | 客戶資料模型：公司 → 服務(1:N，SLA 可客製覆寫) → 聯絡人(姓名*／角色*／email*／通訊 ID／IP) + 我方對接窗口(業務／技術)；含 Issuer OID、地區、客戶狀態(導入中／上線／暫停／終止) | Must | 主管 Q17／Andrew／Diego |
| F6 | 聯絡窗口管理：不設數量上限（約 8–10/家），需 active/inactive 狀態與角色／優先序 | Must | Owner／Diego／Ray |
| F7 | 客戶資料匯入／匯出／批次修改（如業務離職批次換窗口） | Must | 主管／Andrew |
| F8 | 每筆資料記錄新鮮度：last_verified_at／updated_by／verification_status／stale_flag | Should | Diego／Ryan |
| F9 | 客戶資料維護排程：預設半年（可調），定期匯出給客戶確認後匯回更新 | Should | A組衝突4／Owner／主管 |
| **標籤與分類** | | | |
| F10 | 既定標籤清單（先定義再開放操作）：客戶(發卡行／公司)、服務(ACS/3DSS)、角色職責(技術/業務窗口、緊急聯絡人、PG)、問題(交易/系統/公告/維護) | Must | Andrew／Alan／Ryan |
| F11 | SLA 標籤：公版 + 客製，分級(Platinum/Gold/Standard、7x24/5x8)，同客戶多 SLA 取最嚴格 | Must | Andrew／Alan |
| F12 | 資料可信度標籤：查無或資料過舊 → 降可信度，低可信度提示人工驗證 | Should | Diego／Ryan／主管 |
| **AI 介接與治理** | | | |
| F13 | 提供 CLI／API 供 AI Agent 呼叫；CLI 回傳固定格式（AI 可再轉 md/csv/JSON） | Must | baseline（CLI 給 agent 呼叫）／Diego |
| F14 | **AI Agent 權限 = 只讀**；新增／修改／刪除一律經後台人工 Approve | Must | 主管／Jesse／Ryan／Wayne（全體一致） |
| F15 | AI Summary 內容：客戶角色／職位／是否技術人員 + 根因 + 嚴重度 + 是否需立即處理 + 行動建議/步驟 | Must | 主管／Ryan／Sasha |
| F16 | 回覆模板：依問題分類套標準模板（公告／維護／交易固定格式），發送前人工 review | Should | Owner／Diego／主管 |
| **後台與權限** | | | |
| F17 | 後台管理介面：支援 **SSO**、方便人類閱讀、可維護標籤／分流規則／資料 | Must | 主管／Ryan／Wayne |
| F18 | 權限控管：後台角色(Admin/Viewer…) + CLI 權限(誰能呼叫哪些 CLI) + 調用權限(誰能發 API)；RBAC 設計可擴充（保留 Business Viewer 空間） | Must | Ryan／Diego／Wayne／Owner |
| F19 | 操作留痕 Audit Log：後台操作以 **Before/After** 呈現；人工查詢進 DB／API 呼叫寄 log | Must | Jesse／Ryan／Wayne／A組衝突5 |
| **備援** | | | |
| F20 | 工具不可用 → 人工備援：定期（每日/每週）匯出客戶資料至受控位置（GitLab MD／Excel）作唯讀備援，掛掉時人工查 | Must | Owner／Chloe／Diego |
| **監控（SRE 本行）** | | | |
| F21 | 監控：API 頻率／成功率(2xx，<80% 告警)／latency；系統 CPU/mem/disk(90% 告警)；4xx/5xx | Must | SRE 全體 |
| F22 | AI Token 用量監控：第三方組件，超量自動 suspend + 告警設計者 | Must | SRE（Alan/Sasha/Ray/Andrew） |
| F23 | 結構化 Log Summary：含 OID／地區／API 執行時間，供複合排查 | Should | SRE（Sasha/Andrew） |

---

## 2. 非功能需求

> 效能類**必附規模假設**。寫不出規模的先別列。

- **效能**：單筆查詢回應要短（Owner：慢就不如自己查）；重大事件通知名單須快（Owner 底線 5 分鐘內）。
  - 規模假設：約 **300 家客戶**、每家 **8–10 位聯絡窗口**、**觸發式低併發**（非高 QPS 交易系統）。
  - P95 目標數字待壓測後定（SRE：機器開多大要壓測）。
  - ⛔ **Queue／HA 排隊：此規模屬假議題，預設不做（WON'T）**——量化後即蒸發（A組衝突1）。但 SRE 把架構綁在預算上，**預算上限拍板前**保留「Priority Queue + VIP 不走 queue」為備案，僅在規模成長時啟用。
- **安全 / 合規**：
  - **PCI**：不儲存 token／卡號 → 工具可落在 CDE 範圍外（SRE 一致）。
  - **ISO-27001**：公司即將導入；個資存 DB 需加密／遮罩（顯示層遮罩），基礎設施與資料異動須留紀錄、必要時 approve。
  - **個資法**：email／手機／姓名等個資須保護；匯出檔案加密、敏感欄位遮罩。
  - 請求端 **IP 白名單**管控（SRE 一致）。
- **可用性**：
  - 定位為**次要服務**（不影響交易），可接受拉長回應／冷啟動以省成本（SRE）。
  - SLA 超時或取不到資料 → AI 先回**中性罐頭信**（服務降級），或轉真人；AI 內建客戶↔SLA 對應，不必每次回查工具。
  - 工具掛掉 → 唯讀備援 + 人工（見 F20）。
  - SLO 具體數字（可用性／延遲／識別正確率）待定。
- **部署 / 可觀測性**：GCP + K8S 容錯架構、單獨模組部署（SRE 一致；AWS vs GCP 見開放問題）。監控見 F21–F23。

---

## 3. 開放問題 / 待 escalate

> 沒拍板的、牽涉風險的、兩案並陳的——全列，並標「需向誰確認」。

- [ ] **預算上限未定**（最關鍵）：SRE 反覆強調架構／是否需 Queue／機器規模都取決於此，主管尚未給數字 — **待主管拍板（escalate）**。
- [ ] **domain 符合但人不在名單**：自動歸戶 vs 人工防詐騙。建議標 `[待審核_潛在客戶]` + 撈該 domain 既有窗口人工照會，**不自動歸戶**；資安取捨 — **待主管講風險再決定（escalate）**。
- [ ] **部署平台 AWS vs GCP**：SRE 偏熟悉度，主管要「便宜優先」 — SRE 列價格交主管審核（escalate）。
- [ ] **ISO-27001 加密／遮罩範圍**：哪些欄位、儲存層或顯示層、approve 流程門檻 — 待資安／主管（escalate）。
- [ ] **權限分層具體規格**：一層 vs 兩層未定；完整角色清單；CLI 權限與後台角色是否同一套 — 待主管／Owner。
- [ ] **Bulk query 是否納入 MVP**：Owner 要看「所有 ACS 客戶」等分群清單，**主管未意識到此需求** — 待主管裁決範圍。
- [ ] **審核治理不一致**：人改不審、AI 改要審，邏輯不一致 — 建議改用「操作風險分級」決定是否 approve（VIP/刪除/批次覆蓋才強制） — 待主管。
- [ ] **Admin 後台 vs 客戶資訊工具的資料架構**：共用同一 DB，還是兩套 + 人工同步（需顯示同步狀態） — 待架構決定。
- [ ] **歷史問題紀錄**：放 Developer Site，AI Summary 是否需讀取以判斷「重複問題／是否需處理」 — scope 待定。
- [ ] **回應時間「短」的底線**：2s／10s／1min／5min？哪些查詢絕不能慢？VIP 是否不同標準 — 待 Owner 劃線。

---

### 交件

- 檔名：`kyson_w3-需求清單.md`
- 上課前（6/18 前）寄給主管 Ardi（ardichuang@hitrust.com）。
- 同組原料相同沒關係，看個人怎麼收斂。
- 自我檢查：丟給不認識專案的人（或 AI）能照著做嗎？模糊詞已改具體。
