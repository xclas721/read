# 第一階段 email 查詢 Spec — 開發前疑問清單

> 針對 [kyson_w3-spec-第一階段-email查詢.md](kyson_w3-spec-第一階段-email查詢.md)。
> 評估：方向與驗收條件清楚，但要「直接開工寫程式」仍有以下需確認的缺口。
> 分類：🔴 開工前必答（卡住實作）／🟡 影響設計但可先用假設／🟢 細節，可後補。

---

## 1. 資料模型 / DB（🔴 必答）

- [ ] 🔴 **沒有 DB schema**。spec 只給回傳 JSON，沒給資料表結構。需要：customer、contact、service/SLA、domain 允許清單、audit 各表的欄位、型別、主鍵/外鍵、索引（email 查詢要建 index）。
- [ ] 🔴 **email 與客戶的關聯方式**：是靠「聯絡人名單精確比對 email」還是「domain 比對」？T1.2 寫 domain 吻合即判定為客戶 A，但 R1.1 又是「反查單筆客戶」。當 email 不在名單、但 domain 命中時，到底算 found 還是 not_found？（開放問題有提到「不自動歸戶」，但回傳 status 要給哪個？）
- [ ] 🔴 **一個 domain 對多客戶**怎麼處理？例如多家公司共用 `gmail.com`、或集團共用 domain。判定會不會撞？是否需要 domain 唯一性約束？
- [ ] 🟡 **同一 email 出現在多個客戶名單**：回傳哪一筆？報錯還是回多筆？
- [ ] 🟡 `issuer_oid`、`region`、`customer_status` 的**值域與來源**（尤其 customer_status 的中文 enum 是否就是 DB 存的值）。

## 2. 認證 / 授權（🔴 必答）

- [ ] 🔴 **token 是什麼機制**？JWT？OAuth2 client credentials？API key？由誰簽發、如何驗證、放在哪個 header（`Authorization: Bearer`?）。
- [ ] 🔴 **角色（role）從哪來**？T1.8 依角色遮罩，但 role 是 token claim、還是查 DB 對應 caller？有哪些角色（Viewer / 完整 / 其他）、各自可見哪些欄位？
- [ ] 🟡 401 vs 403 的**分界**：spec 說「不洩漏客戶是否存在」，那 token 過期、無此 API 權限、權限不足要分別回哪個？

## 3. API 介面契約（🔴 必答）

- [ ] 🔴 **email 怎麼傳**：query string（`?email=`）還是 header？大小寫、URL encoding、`+` 號等如何處理（T1.1 要求大小寫視為同一筆）。
- [ ] 🔴 **錯誤回傳格式**：`error_code` 的完整清單與對應 HTTP status（400/401/403/405/5xx），以及 error body 的 JSON 結構（spec 只在 JSON schema 提了 `status:error` 但沒定義 error 物件）。
- [ ] 🟡 **`status=error` 何時用**：跟 HTTP 4xx/5xx 的關係？是 200 body 帶 error，還是直接非 200？兩種並存會讓呼叫方混亂。
- [ ] 🟢 是否要 `schema_version` 欄位（開放問題已列，但若要做就得現在決定）。

## 4. 遮罩規則（🟡）

- [ ] 🟡 **遮罩演算法要精確定義**：`王*明`、`a***@a-bank.com` 只是範例。姓名 2 字／4 字、英文名、email local part 長度不同時的規則？是否需要可逆（給高權限角色看明碼）？
- [ ] 🟡 **DB 層加密 vs 應用層遮罩**：T1.8 寫「ISO-27001 導入前至少應用層遮罩」，那第一階段 DB 是**明碼存**還是要欄位加密？這影響 schema 與是否需要 KMS。

## 5. 商業邏輯細節（🟡）

- [ ] 🟡 **「最嚴格 SLA」排序**：Platinum > Gold > Standard 的優先序是否確定？support_hours（7x24 / 5x8）也要取嚴格嗎？
- [ ] 🟡 **`same_domain_contacts` 內容格式**：spec 中是空陣列 `[]`，但 T1.4 要求列出同 domain 既有窗口。每筆要哪些欄位？要不要遮罩？
- [ ] 🟡 **`data_confidence` 判定規則**：high/medium/low 各自的觸發條件？只在「無訂閱服務→low」有定義，其餘呢？
- [ ] 🟢 **`our_contacts`（我方業務/技術窗口）來源**：哪張表、是否一定有值。

## 6. 稽核 Log（🟡）

- [ ] 🟡 **audit 存哪**：同一個 DB 另一張表、還是寫 GCP Cloud Logging？查詢/保存期限要求？
- [ ] 🟡 「token 對應主體」**記錄什麼識別碼**（caller_id？subject？）才能追責又不存敏感資料。

## 7. 非功能 / 環境（🟡 / 🟢）

- [ ] 🟡 **效能目標到底抓 60s 還是 10s**：開放問題說可能收緊到 10s 級。架構（要不要 cache、index 策略）會因此不同，建議現在定一個實作目標。
- [ ] 🟢 **規模假設確認**：300 家客戶、~3000 聯絡人是否就用這個數字做壓測與 index 規劃（spec 標「待 Kyson 確認」）。
- [ ] 🟢 **技術棧**：語言/框架/DB 產品（GCP 上是 Cloud SQL MySQL？Postgres？）spec 只說「DB only、部署 GCP」，沒指定。
- [ ] 🟢 **部署形態**：Cloud Run / GKE / GCE？是否要容器化？

## 8. 範疇確認（🟢）

- [ ] 🟢 **測試資料/種子資料**由誰提供？要不要我自建假資料跑 T1.1~T1.9？
- [ ] 🟢 是否需要 OpenAPI / Swagger 文件作為交付物之一（AI Agent 整合方可能需要）。

---

## 如果只能先問三題（最卡住開工的）

1. **DB schema 與「email→客戶」判定邏輯**：精確比對名單 vs domain 比對，命中/未命中各回什麼 status？（第 1 節）
2. **token / role 機制**：用什麼驗證、role 從哪取、角色↔可見欄位對照表？（第 2 節）
3. **技術棧與 DB 產品**：語言/框架/Cloud SQL 種類，才能起專案骨架。（第 7 節）
