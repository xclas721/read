## 範例：email 查詢這條線，寫到可驗收

**用例**

AI agent 用寄件人 email 查單筆客戶 → 回 JSON；查無 → 提示；無權限 → 拒絕

**功能**

F1 email 查 · F2 回 JSON 固定欄位 · F3 查無回提示

**這條線的非功能（NFR）**

個資 AES 不明碼 · 每查寫 audit · ≤1 分鐘@~3000 筆 · 僅授權 token

NFR 飄在清單 ≠ 寫進 spec——要像右邊這樣織進驗收
