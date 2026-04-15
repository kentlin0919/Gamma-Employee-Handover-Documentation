# Azure 虛擬網路建立與設定指南 (CMUA)

本文件整理 `CMU Alliance` 專案在 Azure 上建立或補齊虛擬網路 (`VNet`) 與子網路 (`Subnet`) 的標準流程。內容以目前 `cmua` 實際環境與既有 VM / SQL 私網連線需求為基準，目標是讓 `ap1`、`ap2`、Application Gateway 與 SQL Private Endpoint 使用同一套可維運的私網架構。

文內擷圖為 2026-04-12 自 Azure Portal 擷取的現場畫面，主要作為辨識頁面與操作位置參考；若 Azure 介面版本更新，請以同名資源與功能區塊為準。

---

## 0. 先看現況再動手

在撰寫這份文件時，專案內較早期的總覽文件仍有 `weigong` 命名示意，但目前 `CMUA` 實際 Azure Portal 中已可看到：

* 應用 VM 為 `cmua-cmu-ap1`、`cmua-cmu-ap2`
* Resource Group 為 `cmua-cmu`
* SQL Server 為 `cmua-cmu-db`
* SQL Private Endpoint 為 `cmua-cmu-network-db`
* VM 掛載的既有虛擬網路顯示為 `cmua-cmu-Network/...`

因此後續若要新增或調整 VNet，請以 **現場既有 `cmua` 網路資源為準**，不要再另外新建一套 `weigong` 命名的 VNet。

## 1. 目標網路架構

`CMUA` 專案建議維持下列分層：

| 區塊 | 建議名稱 | 用途 |
| :--- | :--- | :--- |
| **Virtual Network** | 沿用既有 `cmua` VNet | 專案核心私網骨幹 |
| **AppGatewaySubnet** | `AppGatewaySubnet` | 僅供 Azure Application Gateway 使用 |
| **BackendSubnet** | `BackendSubnet` | 放置 `ap1`、`ap2` 等應用 VM |
| **DataSubnet** | `DataSubnet` | 放置 SQL Private Endpoint 等資料層私網端點 |

原則：

* `AppGatewaySubnet` 不可混放 VM 或 Private Endpoint。
* `BackendSubnet` 只放應用層 VM。
* `DataSubnet` 只放資料層 Private Endpoint，避免和應用 VM 混用。
* 應用 VM 必須能透過 Private DNS / Private Endpoint 解析並連到 `cmua-cmu-db`。

---

## 附錄：Azure 子網路錯誤解析 (Subnet Overlap)

在進行網路擴充或建立 ALB 時，若遇到無法新增子網路的情況，通常與 **CIDR 重疊** 有關。以下為實戰經驗總結：

### 🧠 問題核心：子網重疊 (Subnet Overlap)
當嘗試新增一個子網路（例如 `10.0.0.0/24`）到現有的 VNet 時，若 Azure 報錯：`位址範圍與其他子網路重疊`，表示該區段已被佔用。

*   **關鍵觀念**：子網不能有任何交集，也不能「包含」或「被包含」於其他子網。
*   **CIDR 範例**：
    *   `10.0.0.0/16`：整個 VNet 的大池子。
    *   `10.0.255.0/27`：GatewaySubnet (32 個 IP)，範圍是 `.255.0` 到 `.255.31`。
    *   `10.0.0.0/24`：若此段已被 `cmua-cmu-network` 等資源佔用，則無法再以此網段建立新子網。

### ✅ 避坑直接解法
選用一段完全獨立且未被使用的 IP 區間。
*   **檢查方式**：進入 **VNet -> Subnets** 檢視所有已存在的網段。
*   **選取空位**：例如原本已有 `10.0.0.x` 與 `10.0.255.x`，則可以考慮改用 `10.0.1.0/24`、`10.0.2.0/24` 等完全不重疊的範圍。

### 🧠 觀念心智圖
```text
Azure VNet (10.0.0.0/16)
│
├── Subnet A (10.0.0.0/24) ❌ 已存在
├── Subnet B (10.0.255.0/27) ✔ Gateway
│
└── 新 Subnet ❌ 不能：
     ├─ 重疊 (Overlap)
     ├─ 包含其他子網
     └─ 被包含
👉 必須為「完全獨立」的 IP 區間。
```

---

## 3. 建立 Azure VNet 流程 (後略...)
*(此處保留原本的所有步驟與既有網段值，不作更動)*
