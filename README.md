# 嘉鈊科技：軟體研發與系統運維交接總目錄 (Master Handover Index)

> **本倉庫使命**：為嘉鈊科技 (Gamma Technology) 提供結構化、自動化且易於接軌的技術資產管理中心。確保每一項專案在人員異動或階段轉換時，皆能維持開發的高效能與系統的高穩定性。

---

## 🚀 快速入口 (Quick Access)
- 📋 **[交接與開發維護標準 SOP](./SOP_HANDOVER_GUIDE.md)**：接手新專案的第一步（環境建置、任務管理、部署流程）。
- 🤖 **[AI 交接指引與任務分析 (GEMINI.md)](./GEMINI.md)**：利用 Gemini CLI 進行自動化文件生成與 Redmine 歷史任務深度分析。
- 📁 **[交接文件範本 (00-template)](./projects/00-template/README.md)**：建立新專案交接文件時的標準格式。

---

## 📂 專案組合 (Project Portfolio)

本目錄將所有開發中的專案劃分為五大類別，請點選連結查看詳細的交接文件與技術細節：

### 📱 [01-brymen-app (iombtc APP)](./projects/01-brymen-app/README.md)
*   **階段**：第二階段 (主線) / 第三階段 (開發中)
*   **核心技術**：Flutter, GetX, BLE, Hive, Isolate 大檔案解析。
*   **關鍵優化**：藍牙初始化狀態機、Realtime LiveChart 效能模型、SSR 儲存機制。

### 🌐 [02-infra-k8s-pve (基礎設施)](./projects/02-infra-k8s-pve/README.md)
*   **核心內容**：Proxmox VE 虛擬化平台、Kubernetes (K8s) 叢集配置、Caddy 遷移紀錄。
*   **內部維護**：Redmine Dashboard 開發與監控系統配置。

### ♻️ [03-recycling-3c91 (回收系統)](./projects/03-recycling-3c91/README.md)
*   **核心內容**：3C91 回收自動化、計價引擎邏輯、訂單狀態流轉。
*   **數據分析**：數據儀表板統計條件與資料庫 Schema。

### 🧩 [04-others (跨專案支援)](./projects/04-others/README.md)
*   **包含專案**：
    - **GX-桃園E健康**：SSL 憑證、WebView 跑版修復。
    - **為恭醫院**：初期開發環境建立。
    - **瀚荃空氣偵測器2**：前端 APP 開發。
    - **中華郵政物流園區**：AIoT 虛擬訊號測試。
    - **永彰智慧辦公室**：Mail Server 告警系統。

### 🛠️ [05-tools-rd (研發工具)](./projects/05-tools-rd/README.md)
*   **核心工具**：Gemini CLI 配置、Git Hooks 自動化檢查、App 自動部署 (Fastlane) 腳本。

---

## 🛠️ 如何使用本倉庫 (How to Use)

### 1. 針對接手人員 (For Successors)
- **第一小時**：閱讀根目錄的 [SOP 指南](./SOP_HANDOVER_GUIDE.md)。
- **第一天**：根據 [GEMINI.md](./GEMINI.md) 中的任務分析，了解 Redmine 上指派給您的核心技術債。
- **第一週**：依照專案資料夾下的 `docs/ARCHITECTURE.md` 熟悉程式碼結構。

### 2. 針對現有開發者 (For Maintainers)
- **同步更新**：每完成一個重大 Redmine 任務，請同步更新對應專案的 `README.md`。
- **AI 輔助**：可使用 `gemini` 指令自動分析最新 Commit 並產出技術摘要填入文件中。

---

## 💡 交接三大原則 (Handover Principles)
1.  **可重現性 (Reproducibility)**：接手者應能在一小時內完成本地環境建置。
2.  **透明性 (Transparency)**：所有手動修改與「隱藏知識」必須文件化。
3.  **安全性 (Security)**：嚴禁將明文密碼寫入文件，使用指定的金鑰管理工具。

---
*嘉鈊科技 (Gamma Technology) - 內部機密文件*
