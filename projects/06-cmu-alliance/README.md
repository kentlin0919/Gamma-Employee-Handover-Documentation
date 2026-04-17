# 06-cmu-alliance: 中醫大聯盟 (CMU Alliance)

本專案目錄存放與「中醫藥大學策略聯盟醫院」相關的雲端基礎設施與系統架構文件。

## 專案概觀

*   **專案名稱**: 中醫藥大學策略聯盟醫院 (CMU Alliance)
*   **Redmine 專案 ID**: [79](https://redmine.viuto-aiot.com/projects/gx_cmuh_p1) (GX-中醫藥大學策略聯盟醫院-2026系統階段)
*   **主要醫院對象**: 為恭醫院、中國醫藥大學附設醫院 (總院)、安南醫院、北港醫院、兒童醫院、亞大附醫等。

## 目錄結構

*   `docs/`: 核心架構文件與雲端資源管理規範。
    *   `PROJECT_INFO.md`: 專案概觀、站台建置資訊與維運流程 (同步自 Redmine Wiki)。
    *   `AZURE_CLOUD_RESOURCES.md`: Azure 雲端資源管理維運手冊。
    *   `procedures/`: 標準作業程序。
        *   `AZURE_SQL_SETUP_GUIDE.md`: Azure SQL Database 建立與配置指南。
*   `assets/`: 相關架構圖、截圖與設計資產。

## 關鍵技術棧

*   **雲端平台**: Microsoft Azure
*   **網路架構**: VNet/Subnet, Application Gateway, VPN Gateway, Private Link.
*   **資料庫**: Azure SQL Server, Elastic Pool.
*   **安全認證**: Managed Identity, Key Vault.

---

## 🚀 Azure 基礎設施架設指南 (Infrastructure Setup Guide)

本節提供於 Azure 上從零建置 CMU Alliance 基礎設施的流程導引。請依序完成以下階段之架設，並參考對應的 SOP 文件。

### 第一階段：基礎網路與安全 (Foundation)
- [ ] 建立專案資源群組 (RG)：`cmua-cmu`
- [ ] 配置虛擬網路 (VNet/Subnet)：劃分前端、後端與資料層子網。
  - 👉 參考文件：[Azure 虛擬網路建立與設定指南 (CMUA)](./docs/procedures/AZURE_VNET_SETUP_GUIDE.md)
- [ ] 建立 Key Vault：存放環境祕密與憑證。

### 第二階段：資料持久層 (Data Layer)
- [ ] 建立 Azure SQL Server 與資料庫 (PaaS)。
- [ ] 設定 Private Link / Private Endpoint：確保資料庫僅限內網存取。
  - 👉 參考文件：[Azure SQL Database 建立與配置指南](./docs/procedures/AZURE_SQL_SETUP_GUIDE.md)

### 第三階段：應用伺服器 (Application Layer)
- [ ] 建立虛擬機器 (VM)：部署 `ap1` 與 `ap2` 生產環境伺服器。
- [ ] 設定 Managed Identity：授予伺服器存取 Key Vault 的權限。
  - 👉 參考文件：[Azure VM 建立與配置指南 (CMUA)](./docs/procedures/AZURE_VM_SETUP_GUIDE.md)

### 第四階段：流量管理與存取控制 (Traffic & Access)
- [ ] 部署 Application Gateway (v2 + WAF)：作為公網流量入口。
  - 👉 參考文件：[Azure Application Gateway (ALB) 設定指南](./docs/procedures/AZURE_CMUA_ALB_SETUP_GUIDE.md)
- [ ] 配置 VPN Gateway：建立 P2S 隧道供管理者遠端維運。

### 第五階段：儲存與外部整合 (Storage)
- [ ] 建立 Storage Account 與檔案共用 (File Share)。
- [ ] 設定 SFTP 存取流程（若有外部對接需求）。
  - 👉 參考文件：[SFTP 流程架設指南](./docs/procedures/SFTP_E2E_SETUP_GUIDE.md)

---
*此文件為嘉鈊科技交接文件之一部分。*
