# W3 課前作業 — 需求詳細清單（Kyson · 精簡版）

> 精簡版：聚焦 Must、合併細節，方便快速閱讀與課堂使用。完整版見 [kyson_w3-需求清單.md](kyson_w3-需求清單.md)。
> 工具：**客戶資訊工具**（AI Agent 的基礎識別層）。視角：SRE（Kyson）。

**範疇邊界**：本工具只做身分識別 + 客戶/聯絡人/服務/SLA 查詢與維護 + AI Summary + CLI/API；歷史問題分析、對外寄信屬其他工具。

---

## 1. 功能需求（Must 為主）

| ID | 需求描述 | MoSCoW | 來源 |
|---|---|---|---|
| F1 | 反查單筆客戶 email／基本資料 | Must | baseline（最高頻） |
| F2 | 依 email domain／通訊群組名辨識客戶，並配對服務（ACS／3DSS） | Must | 主管／Andrew |
| F3 | 查無／不在名單 → 提示 + 轉人工 + 標「待審核」，**不自動歸戶**（防詐騙） | Must | Owner／baseline |
| F4 | 客戶資料模型：公司→服務(含 SLA)→聯絡人(姓名*／角色*／email*)＋我方窗口；支援匯入／匯出 | Must | 主管／Andrew／Diego |
| F5 | 聯絡窗口管理：不設上限，需 active／inactive 狀態 | Must | Owner／Ray |
| F6 | 既定標籤清單（客戶／服務／角色職責／問題）＋ SLA 分級（多 SLA 取最嚴格） | Must | Andrew／Alan |
| F7 | 提供 CLI／API 供 AI Agent 呼叫，回傳固定格式 | Must | baseline（CLI 給 agent 呼叫） |
| F8 | **AI Agent 只讀**；新增／修改／刪除一律經後台人工 Approve | Must | 全體訪談一致 |
| F9 | AI Summary：客戶角色／是否技術人員＋根因＋嚴重度＋是否需立即處理＋行動建議 | Must | 主管／Ryan |
| F10 | 後台管理介面 ＋ SSO ＋ 權限分層（後台角色／CLI 權限／誰能發 API） | Must | 主管／Ryan／Wayne |
| F11 | 操作留痕：後台異動以 Before／After 呈現；查詢留痕 | Must | Jesse／Ryan |
| F12 | 工具不可用 → 定期匯出唯讀備援（GitLab／Excel），人工查 | Must | Owner／Chloe |
| F13 | 監控：API 成功率(<80% 告警)／latency／系統資源；AI token 超量自動 suspend + 告警 | Must | SRE 全體 |

> Should（次優先，完整版有）：資料新鮮度欄位、維護排程（半年可調）、資料可信度標籤、回覆模板、結構化 Log Summary。

---

## 2. 非功能需求

- **效能**：單筆查詢別比人工自查慢；重大事件通知名單 ≤ 5 分鐘。規模假設：約 300 家客戶、每家 8–10 窗口、觸發式低併發。⛔ Queue／HA 不做——此規模屬假議題。
- **安全／合規**：不存 token／卡號（避開 PCI）；ISO-27001 導入後個資加密／遮罩、異動留紀錄 + approve；個資法；請求端 IP 白名單。
- **可用性**：定位次要服務，可拉長回應／冷啟動省成本；SLA 超時 → AI 先回中性罐頭信或轉真人；掛掉 → 唯讀備援。
- **部署**：GCP + K8S 容錯、單獨模組。

---

## 3. 開放問題 / 待 escalate

- [ ] **預算上限**（最關鍵）：決定架構與是否需 Queue／機器規模 — 待主管。
- [ ] **domain 符合但人不在名單**：自動歸戶 vs 人工防詐騙 — 待主管講風險。
- [ ] **部署平台 AWS vs GCP** — 待主管看價格。
- [ ] **ISO-27001 加密／遮罩範圍與 approve 流程** — 待資安／主管。
- [ ] **Bulk query（看所有 ACS 客戶等分群）是否納入 MVP** — 主管未意識，待裁決。

---

### 交件

- 檔名：`kyson_w3-需求清單.md`（交件用完整版或本精簡版皆可）
- 上課前（6/18 前）寄給主管 Ardi（ardichuang@hitrust.com）。
- 規模數字 300 家來自 A 組簡報，尚待確認。
