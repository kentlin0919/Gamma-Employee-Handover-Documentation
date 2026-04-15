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
*此文件為嘉鈊科技交接文件之一部分。*
