# 客戶資訊工具 — 衝突收斂簡報

> 2026/06/11 · 報告用 · 每個衝突點 = 衝突 + 各方 Why + MoSCoW 裁決

---

## 衝突 1：回應速度 vs 成本／排隊
- **衝突**：Owner 要快（最慢 1 分鐘）vs SRE 想拉長 latency、用 Queue 省成本。
- **各方 Why**：Owner＝第一線體驗，太慢不如自己查；SRE＝成本受限、保服務不中斷；主管＝省成本、且 300 客戶量根本不會慢。
- **MoSCoW**：⛔ **WON'T** — Queue 不做（假議題，1000 同時也撐得住）；回應速度 1 分鐘內可接受。

## 衝突 2：未知客戶 domain vs CC 兩層信任
- **衝突**：Ardi 只檢查寄件人 domain vs Owner 想再加「CC 人」檢查。
- **各方 Why**：Ardi＝省人工（客戶離職率高、新人多）；Owner＝防詐騙，怕仿冒者觸發 AI 變更權限的操作。
- **MoSCoW**：🔴 **MUST**＝寄件人 domain 檢查（首要）；⚪ **COULD**＝CC 人檢查（先做寄件人，未來有心力再加）。

## 衝突 3：單筆 vs 批次查詢
- **衝突**：原設計只想單筆 vs Owner 要批次（公司全員／有效合約清單／單一聯絡人）+ API 供 AI 對話框呼叫。
- **各方 Why**：Owner＝重大事件要快速撈清單、避免多系統維護；主管＝沒特別想法，需要什麼查詢條件就加。
- **MoSCoW**：🔴 **MUST**＝單筆查詢（查 email 最高頻）；🟡 **SHOULD**＝批次查詢 + API；CLI 固定格式、AI 再轉 md/csv。

## 衝突 4：客戶資料維護頻率（每季 vs 每半年）
- **衝突**：主管原想每季匯出 vs Owner 認為半年即可。
- **各方 Why**：主管＝資料新鮮、避免過期誤判；Owner＝實務上半年夠、不想太常打擾客戶。
- **MoSCoW**：🟡 **SHOULD**＝維護排程，**預設半年、可調**（已對齊）；合約因誤寄解約客戶事件升第一階段。

## 衝突 5：操作紀錄 / 個資治理（人改 vs AI 改 + 查詢要不要記）
- **衝突**：Wayne 認為查詢不用記（怕塞爆 DB）vs SRE[Alan] 認為 ISO 稽核、涉個資，查詢也要留痕。
- **各方 Why**：Wayne＝省 DB；SRE＝可稽核「誰查了誰的個資」、個資不可明碼；主管＝折衷，人工查詢進 DB、API 呼叫寄 log。
- **MoSCoW**：🔴 **MUST**＝查詢留痕（**人工進 DB／API 寄 log**）+ 個資（email/手機/姓名）**加密不明碼**；AI 不信任 domain 才需 approve。

## 衝突 6：AI 自動新增 IP 白名單（防火牆）
- **衝突**：Ryan 想 AI 自動串接 AWS 加白名單、減少工作量 vs Ardi 認為違反 PCI。
- **各方 Why**：Ryan＝省工自動化；Ardi＝PCI 要可追責（執行者／approve 背鍋俠／主管），自動化無人可扛。
- **MoSCoW**：⛔ **WON'T** — 捨棄此功能（違反 PCI、無操作人與負責人）。

## 衝突 7：部署平台 GCP vs AWS
- **衝突**：Ardi 偏 GCP（成本）／Owner 要同一平台 vs SRE＋客服偏 AWS。
- **各方 Why**：Ardi＝GCP CUD 便宜；Owner＝單一平台好管理；SRE＝AWS 操作熟、整合好、省時間；客服＝AWS 查詢直覺好用。
- **MoSCoW**：⏳ **HOW/WHERE 未定** — SRE 將列 **AWS/GCP 價格**，交主管審核是否採 AWS。

---

## MoSCoW 一頁總表

| 級別 | 項目 |
|------|------|
| 🔴 MUST | 單筆查詢(查 email)、寄件人 domain 檢查、查詢留痕(DB/log 分流)、個資加密不明碼 |
| 🟡 SHOULD | 批次查詢 + API、CLI 固定格式轉 md/csv、維護排程(半年可調)、合約狀態防誤寄 |
| ⚪ COULD | CC 人檢查（未來再加） |
| ⛔ WON'T | Queue／排隊、AI 自動改防火牆加 IP 白名單 |
| ⏳ 未定 | 部署平台 GCP vs AWS（待主管看價格定） |
