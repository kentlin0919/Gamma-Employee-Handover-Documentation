# GX-BRYMEN APP 第二階段 (iombtc APP) - 專案交接文件

> **文件資訊**
> - **專案 ID (Redmine):** 75
> - **父任務 ID:** [#10597](http://redmine.gamma.com.tw/issues/10597)
> - **最後更新日期:** 2026-04-08
> - **交接狀態:** 交接準備中

---

## 1. 專案概述 (Project Overview)

本專案為 Brymen 藍牙電表監控 App (專案名稱: `iombtc`)。

### 技術棧 (Tech Stack)

- **Framework:** Flutter (SDK: `>=3.3.0 <4.0.0`)
- **Version:** `1.1.2+54`
- **State Management:** `get: ^4.7.3` (GetX)
- **Local DB:** `hive: ^2.2.3` (Hive Flutter)
- **Communication:** `flutter_reactive_ble: ^5.4.0` (Reactive BLE)
- **Charting:** `fl_chart: ^0.68.0`
- **Logging:** `talker_flutter`, `logger`, `flutter_logs`
- **Background Task:** `flutter_foreground_task: ^8.17.0` (處理 iOS/Android 背景執行)
- **Email Service:** `mailer: ^6.6.0` (發送 Live Chart 報告)

---

## 2. 核心模組說明 (Core Modules)

### 2.1 藍牙通訊與初始化 (BLE Initialization)

- **套件:** 使用 `flutter_reactive_ble` 進行非同步流處理。
- **現況:** 針對 BM109x 等新裝置，初始化流程需優化 (Ref: #11711)。

### 2.2 Data Logger 與背景執行

- **背景任務:** 依賴 `flutter_foreground_task` 維持連線。iOS 跳轉後台停止問題 (#11585) 需檢查此套件之實作。
- **螢幕恆亮:** 使用 `wakelock_plus: ^1.3.2` 確保記錄時螢幕不休眠。

### 2.3 檔案管理與匯出 (Export System)

- **支援格式:** CSV (`csv: ^6.0.0`), PDF (`pdf: ^3.11.0`)。
- **效能優化:** 大檔案解析應透過 `hive` 緩存與 `Isolate` 處理。

---

## 3. 開發環境建置 (Getting Started)

### 3.1 Flutter 版本要求

確保環境符合 `sdk: ">=3.3.0 <4.0.0"`。

### 3.2 產製代碼 (Generated Files)

專案使用 Hive，修改 Schema 後需執行：

```bash
flutter pub run build_runner build --delete-conflicting-outputs
```

### 3.3 資產配置

- 包含大量 `assets/sign/` 下的符號圖示（A, C, E, F, G, H, J, N, D 類別）。
- 字體使用 `Inter`。

---

## 4. 待處理與已知問題 (Pending Issues & Tech Debt)

以下為目前 Redmine 中指派給 Kent 的待處理事項：

| Issue ID | 主題 | 優先級 | 備註 |
| :--- | :--- | :--- | :--- |
| #11711 | 優化 APP 初始化流程 (BM109x) | 正常 | 指令相容性問題 |
| #11588 | Local Time 時區問題修正 | 正常 | UTC 轉換邏輯統一 |
| #11586 | MAX/MIN 統一標記顏色 | 正常 | UI 規格統一 |
| #11478 | 補全 Error Code 18, 19 訊息 | 正常 | UI 提示補全 |

---

## 5. 維護操作手冊

- **CSV 匯出:** 確保 `metricPrefix` 正確套用至圖表 Y 軸 (#11474)。
- **RTC 設定:** `setRTC` 失敗時應有 UI 提示，且 weekday 範圍為 0-6 (#11476)。

---

## 6. 歷史開發軌跡與技術貢獻 (History & Contributions)

本專案經歷了多個階段的迭代，以下記錄過去一年的核心架構演進，供接手者參考設計脈絡：

### 第一階段 (GXxBM-IoMAPP)

- **圖表引擎優化**: 深度客製化 Realtime LiveChart，引入 `xVal` 偏移累加器以配合「位移點位（delta points）」平移模型，並重構 tooltip 觸控提示。
- **藍牙穩定性**: 實作了長距離斷線重連機制、Checksum 錯誤過濾，以及 OTA 韌體更新的防呆流程。
- **記憶體管理**: 實作了記憶體不足 (OOM) 的監控與提前警告機制。

### 第二階段 (目前主力)

- 參見上方「核心模組說明」。主要將 File Manager 拆分為多個 Logic 與 Isolate 架構，大幅提升大檔案解析效能。

### 第三階段 (準備中)

- **DataLogger 對齊**: 操作邏輯需參照 Desktop 方式修改（如 Apply/Delete 狀態切換，監聽 `Status Flag0.bit3`）。
- **MAX/MIN 優化**: 更新讀值比較邏輯與全域標記圖標。

---
#### 備註 (Remarks)

*嘉鈊科技 (Gamma Technology) - 內部機密文件*
