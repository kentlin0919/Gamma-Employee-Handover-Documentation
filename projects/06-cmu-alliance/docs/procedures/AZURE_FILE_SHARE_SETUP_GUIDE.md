# Azure Files SMB 私人端點掛載 — 完整詳細流程

本文件記錄從零開始建立 Azure Files SMB 儲存體並透過私人端點 (Private Endpoint) 安全掛載至 Windows VM 的標準流程。

---

## 前置確認

| 項目 | 說明 |
|------|------|
| VM 與 Storage Account | 建議在同一個訂閱、同一個區域（例如 Japan East）|
| VM 所在 VNet | 必須與私人端點掛在同一個 VNet |
| 作業系統 | Windows Server 2012 以上 |

---

## Step 1：建立 Storage Account

1. Azure Portal 左上角搜尋「儲存體帳戶」→ 點「+ 建立」
2. 填入：
   - 訂用帳戶：選你的訂閱
   - 資源群組：選或新建（例如 `cmua-cmu`）
   - 儲存體帳戶名稱：全小寫英數，全球唯一（例如 `cmuacmu`）
   - 區域：與 VM **相同區域**（例如 Japan East）
   - 效能：標準
   - 備援：依需求（ZRS 適合生產環境）
3. 點「檢閱 + 建立」→「建立」

---

## Step 2：建立 File Share（檔案共用）

1. 進入剛建立的 Storage Account
2. 左側選單「資料儲存」→「檔案共用」
3. 點「+ 檔案共用」
4. 填入：
   - 名稱：例如 `cmuacmu`
   - 存取層：`交易最佳化`（一般用途）或 `熱` / `冷`
   - 配額：依需求（例如 1024 GiB，最大可設 102400 GiB）
5. 點「建立」

---

## Step 3：關閉公用網路存取

> 讓 Storage Account 只能透過私人端點存取，避免從公網直接打進來

1. 在 Storage Account 左側選單 →「安全性 + 網路」→「**網路**」
2. 「公用存取」頁籤 → `Public network access` 點「Manage」
3. 選「**已停用（Disabled）**」
4. 點「儲存」

---

## Step 4：建立私人端點

1. 在 Storage Account 左側「網路」→「**私人端點**」分頁
2. 點「+ 私人端點」

### 4-1 基本
| 欄位 | 填入值 |
|------|--------|
| 資源群組 | 與 Storage Account 相同（例如 `cmua-cmu`）|
| 名稱 | 自訂（例如 `cmua-cmu`）|
| 網路介面名稱 | 自動填入（例如 `cmua-cmu-nic`）|
| 區域 | 與 Storage Account 相同（例如 Japan East）|

### 4-2 資源
| 欄位 | 填入值 |
|------|--------|
| 訂用帳戶 | 你的訂閱 |
| 資源類型 | `Microsoft.Storage/storageAccounts`（自動帶入）|
| 資源 | `cmuacmu`（自動帶入）|
| **目標子資源** | **`file`**（Files SMB 必選此項；若是 Blob 選 `blob`）|

### 4-3 虛擬網路
| 欄位 | 填入值 |
|------|--------|
| 虛擬網路 | VM 所在 VNet（例如 `cmua-cmu-Network`）|
| 子網路 | VM 所在子網路（例如 `cmua-cmu-network`）|
| 私人 IP | 動態配置（預設即可）|

### 4-4 DNS（重要！）
| 欄位 | 填入值 |
|------|--------|
| 與私人 DNS 區域整合 | **是**（必選，否則 DNS 不會解析到私人 IP）|
| 私人 DNS 區域 | 自動填入 `(新) privatelink.file.core.windows.net` |

---

## Step 5：取得 Storage Account Key

1. Storage Account → 左側「安全性 + 網路」→「**存取金鑰**」
2. 點「顯示金鑰」
3. 複製 **key1** 的 Key 值（長串 Base64 字串）

---

## Step 6：在 VM 上確認 DNS 解析

RDP 或 SSH 進入 VM，在 PowerShell 執行：

```powershell
Resolve-DnsName cmuacmu.file.core.windows.net
```

**正確結果**：回傳 `10.x.x.x` 私人 IP  
**錯誤結果**：回傳 `52.x.x.x` 等公網 IP → 等幾分鐘 DNS 傳播再試

---

## Step 7：存入憑證

```powershell
# 先清除舊的
cmdkey /delete:"cmuacmu.file.core.windows.net"

# 存入正確格式憑證
# user 格式必須是 Azure\<storage-account-name>
cmdkey /add:"cmuacmu.file.core.windows.net" `
       /user:"Azure\cmuacmu" `
       /pass:"<Step 5 取得的 Key>"
```

---

## Step 8：掛載磁碟機

```powershell
New-PSDrive -Name Z -PSProvider FileSystem `
            -Root "\\cmuacmu.file.core.windows.net\cmuacmu" `
            -Persist
```

---

## Step 9：設定開機自動掛載（永久）

建議同時執行以下指令以確保持久化：

```cmd
net use Z: \\cmuacmu.file.core.windows.net\cmuacmu /persistent:yes
```

---

## 常見錯誤對照表

| 錯誤訊息 | 原因 | 解決 |
|----------|------|------|
| `Access is denied` | Public access 被關閉或 DNS 未生效 | 確認 DNS 解析到私人 IP |
| `Access is denied`（DNS 正確）| 憑證 user 格式錯誤 | 改用 `Azure\cmuacmu` |
| DNS 回傳公網 IP | 私人 DNS Zone 未連結到 VNet | 確認 DNS Zone 連結狀態 |
| `Unable to reach port 445` | 防火牆擋住 Port 445 | 使用私人端點走內部網路即可解決 |

---

## 整體架構示意

```
VM (cmua-cmu-Network / 10.0.0.x)
    │
    │  DNS 解析 cmuacmu.file.core.windows.net → 10.0.0.y（私人 IP）
    │
    ▼
私人端點 cmua-cmu (10.0.0.y)  ←→  Storage Account cmuacmu（Public 關閉）
    │
    └── File Share: cmuacmu（掛載為 Z:）
```
