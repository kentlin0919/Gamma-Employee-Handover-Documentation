# GEMINI.md

## 統一使用 繁體中文回覆

## 請先使用網路搜尋相關知識後再分析

## Project Overview

This repository, **Gamma-Employee-Handover-Documentation (嘉鈊科技交接文件)**, serves as a central hub for storing and organizing documentation related to various projects within Gamma (嘉鈊科技). Its primary purpose is to facilitate a smooth handover process for employees by providing essential information, configurations, and knowledge about the company's technical infrastructure and software projects.

The project is currently in its early stages, with a directory structure designed to categorize documentation by project:

*   **01-brymen-app:** Documentation for the Brymen application.
*   **02-infra-k8s-pve:** Information about infrastructure, Kubernetes (K8s), and Proxmox VE (PVE).
*   **03-recycling-3c91:** Documentation for the 3C91 recycling project.
*   **04-others:** Miscellaneous documentation and other smaller projects.
*   **05-tools-rd:** Information and tools related to research and development.

## Directory Structure

*   `assets/`: Intended for storing images, diagrams, and other multimedia assets used in the documentation.
*   `projects/`: The main directory for project-specific documentation, further sub-categorized as listed above.
*   `README.md`: Provides a high-level overview of the repository and its contents.
*   `.gemini/`: Contains configuration files for Gemini CLI.

## Key Files

*   **README.md:** This file provides the initial introduction and lists the projects covered by the handover documentation.

## Usage

This repository is intended to be used as a reference for employees who are taking over projects or need to understand existing systems within Gamma. As documentation is added, employees should:

1.  **Navigate to the relevant project directory** within `projects/`.
2.  **Read the Markdown files** to understand the project's architecture, configurations, and maintenance procedures.
3.  **Refer to the `assets/` directory** for any supporting visual information.

Future contributors should ensure that documentation is kept up-to-date and follows a clear, concise format to maximize its usefulness for the handover process.

## 軟體開發交接指引 (Software Handover Guidelines)

為了確保嘉鈊科技的專案在人員異動時能無縫接軌，本章節提供針對 Redmine 任務分析與交接文件撰寫的標準建議。

### 1. Redmine 任務現況分析 (以 iombtc APP 為例)

根據目前的 Redmine 狀態 (專案 ID: 75)，以下是接手人員應優先關注的核心技術債與未竟事宜：

*   **App 初始化與連線邏輯 (#11711)**: 目前針對 BM109x 等新裝置在連線後的初始化流程有潛在 Bug (如 `GetBleDataOutputInterval` 指令衝突)，接手者需重新梳理藍牙初始化狀態機。
*   **Data Logging 核心機制 (#11592, #11587)**: 包含預約模式 (Scheduling) 與即時模式的切換邏輯，以及 endtime 更新的時效性問題。
*   **時區處理 (#11588)**: 系統內部 UTC 轉 Local Time 的一致性問題，需確保檔案管理與圖表顯示的時區處理邏輯統一。
*   **iOS 平台穩定性 (#11585, #11479)**: 處理 iOS 於 App 跳轉後的後台執行限制，以及建置環境中的 messages.g.h 與套件還原問題。
*   **架構重構紀錄 (#11477)**: 檔案管理模組已初步拆分為 `DataLoaderLogic`、`ChartProcessorLogic` 與 `ExportLogic`，接手者應優先閱讀此部分的重構文件。

### 2. 交接文件撰寫範本 (Handover Template)

一份高品質的交接文件應包含以下核心模組：

#### A. 專案概觀 (Project Overview)
*   **開發目標**：解決什麼問題？(例如：iombtc APP 為藍牙電表監控工具)
*   **關鍵連結**：Figma 設計稿、API Swagger 文件、Redmine 父任務 ID。

#### B. 開發環境建置 (Getting Started)
*   **前置工具**：(例如：Flutter SDK 版本、CocoaPods 版本)
*   **環境變數 (.env)**：說明各項 API Key 或 Secret 的取得來源 (嚴禁將明文密碼寫入交接文件)。
*   **啟動指令**：`git clone` -> `pub get` -> `pod install` -> `run` 的標準流程。

#### C. 技術架構 (Technical Architecture)
*   **技術棧**：使用的框架 (Flutter)、資料庫 (Hive/SQLite)、通訊協定 (BLE)。
*   **目錄結構**：解釋 `src/` 下的核心目錄意義，例如 `services/` 負責藍牙通訊，`providers/` 負責狀態管理。
*   **核心邏輯流程**：建議使用 Mermaid 流程圖描述「連線流程」或「資料落庫流程」。

#### D. 已知問題與技術債 (Known Issues)
*   **待解決 Bug**：直接連結至 Redmine Issue ID。
*   **暫時性的 Hack**：解釋為什麼當初這樣寫，以及建議的重構方向。

#### E. 關鍵維護操作 (Maintenance Tasks)
*   如何更換 iOS/Android 的簽章？
*   如何發佈新的版本到 TestFlight 或 Play Store？

### 3. 交接三大原則
1.  **可重現性**：接手者必須能在 1 小時內成功在本地端跑起開發環境。
2.  **透明性**：所有手動執行的動作 (例如：手動修改某個 Podfile) 必須文件化。
3.  **安全性**：確保所有敏感金鑰皆儲存在公司指定的密碼管理工具中。

