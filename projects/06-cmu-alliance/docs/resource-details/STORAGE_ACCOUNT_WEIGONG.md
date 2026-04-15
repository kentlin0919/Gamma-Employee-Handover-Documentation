# Azure 資源明細：Storage Account (weigong)

本文件詳細記錄 `weigong` 儲存體帳戶的組態設定，主要用於存放專案相關檔案與應用程式資料。

## 1. 基本資訊

| 項目 | 值 |
| :--- | :--- |
| **名稱** | `weigong` |
| **資源群組** | `weigong` |
| **位置** | `Japan East (japaneast)` |
| **帳戶種類** | `StorageV2 (一般用途 v2)` |
| **複寫** | `區域備援儲存體 (ZRS)` |
| **建立時間** | `2026/03/23` |

---

## 2. 網路安全設定 (關鍵)

> ⚠️ **重要限制**：本帳戶已**完全停用公用網路存取**。

*   **公用網路存取**：`Disabled` (已停用)。
*   **私人端點 (Private Endpoint)**：
    *   **名稱**：`weigong`
    *   **掛載子網路**：`weigong/web-subnet` (IP: `10.0.1.0/24`)。
    *   **DNS 解析**：透過 Private DNS Zone `privatelink.file.core.windows.net` 進行內部解析。
*   **連線要求**：只有位於 `web-subnet` 或具備 VNet 路由權限的資源（如 `ap1`, `ap2` VM）才能存取。

---

## 3. 檔案服務設定 (Azure File Share)

*   **檔案共用名稱**：`weigong`
*   **存取層**：`交易最佳化`
*   **配額**：`100 TiB`
*   **資料保護**：
    *   **虛刪除 (Soft Delete)**：已啟用（保留 14 天）。
    *   **SMB 加密**：傳輸中要求加密（Requirement for secure transfer: Enabled）。

---

## 4. 安全性與加密

*   **傳輸安全**：強制要求 TLS 1.2 以上版本。
*   **金鑰存取**：已啟用「儲存體帳戶金鑰存取」。
*   **加密類型**：Microsoft 管理的金鑰 (MMK)。
*   **身分驗證**：目前未設定 AD DS / Azure AD DS 型存取。

---

## 5. 存取金鑰維護

*   **Key1 / Key2**：上次輪替於 `2026/03/23`。
*   **建議**：金鑰應定期輪替（建議每 90 天一次），並將新金鑰更新至相關應用程式的 App Settings 或 Key Vault。

---

## 6. 維運常見問題 (Troubleshooting)

| 問題 | 可能原因 | 解決方法 |
| :--- | :--- | :--- |
| **無法從本機連線** | 公網已停用 | 必須透過 VPN 連入 VNet，或在 VM 上操作。 |
| **連線逾時** | DNS 解析失敗 | 檢查 `privatelink.file.core.windows.net` 區段是否已連結至 VNet。 |
| **Permission Denied** | 金鑰過期或錯誤 | 重新從 Portal 取得 Key1 並更新連線字串。 |
| **誤刪除檔案** | 人為操作 | 在 14 天內可透過「虛刪除」功能進行復原。 |
