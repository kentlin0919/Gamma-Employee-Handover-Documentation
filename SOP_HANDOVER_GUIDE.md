# 嘉鈊科技：專案交接與開發維護 SOP (Standard Operating Procedure)

本指南旨在提供接手開發者一套標準的操作流程，確保能快速進入開發狀態並維持系統穩定。

---

## 1. 環境初始化流程 (Environment Setup)

無論接手哪一個專案，請務必依照以下步驟檢查開發環境：

### 1.1 Flutter 專案 (如 Brymen APP)
1. **版本檢查**：查看 `pubspec.yaml` 中的環境限制，建議使用 `fvm` (Flutter Version Management) 管理版本。
2. **依賴還原**：
   ```bash
   flutter pub get
   ```
3. **iOS 平台特定處理**：
   - 進入 `ios/` 目錄執行 `pod install`。
   - 若遇到 `messages.g.h` 遺失，請執行 `flutter pub run pigeon` (依專案配置)。
   - 確保 `wakelock_plus` 等套件維持在穩定版本，避免建置失敗。
4. **Android 平台特定處理**：
   - 檢查 `android/app/proguard-rules.pro` 是否已載入混淆規則。

### 1.2 K8s 與基礎設施
1. **權限申請**：向資訊部申請 VPN 權限以存取 PVE 內網。
2. **工具安裝**：安裝 `kubectl` 與 `lens` (或類似 K8s Dashboard)。
3. **連線驗證**：
   ```bash
   kubectl config use-context gamma-prod
   kubectl get pods -n prod
   ```

---

## 2. Redmine 任務接軌流程 (Task Management)

接手者應每天執行以下動作：

1. **檢查指派清單**：進入 Redmine 篩選「指派給我」且狀態為「新建立」的任務。
2. **閱讀歷史記錄**：
   - 優先查看「備註」中的討論紀錄。
   - 查找關聯的 Git Commit (格式：`Refs #IssueID`) 以理解程式碼變更脈絡。
3. **工時填報**：
   - 每天下班前，紀錄當天花費的時數與活動 (開發/除錯/測試)。
   - 若任務已完成，請將狀態改為「已解決」並指派給測試人員 (如 Chris Chen)。

---

## 3. 程式碼開發與 Commit 規範 (Development Workflow)

### 3.1 分支策略
- `main` / `master`：正式發佈分支。
- `develop`：主要開發分支。
- `feature/task-#IssueID`：新功能或 Bug 修正分支。

### 3.2 Commit 格式
必須包含 Redmine Issue ID，範例如下：
```text
feat(ble): 優化 BM109x 初始化連線流程

- 修正 GetBleDataOutputInterval 指令衝突
- 增加連線逾時處理邏輯

Refs #11711
```

---

## 4. 測試與驗證流程 (Validation)

### 4.1 自動化測試
- 執行 `flutter test` 確保核心邏輯（如時區轉換、解碼器）無誤。

### 4.2 手動驗證清單 (以 Brymen 為例)
- **連線測試**：切換不同電表型號 (BM109x, BLE8x5x) 測試初始化是否卡住。
- **背景測試**：啟動 Realtime Logger 後，將 App 切換至後台 5 分鐘，確認記錄未中斷。
- **時區測試**：將手機系統時區改為 UTC+0 或其他時區，確認檔案管理清單顯示時間正確。

---

## 5. 部署與發佈流程 (Deployment)

### 5.1 Android 發佈
1. 執行 `flutter build apk --release`。
2. 上傳生成的 APK 至公司內部分發平台或 Play Store。

### 5.2 iOS 發佈
1. 使用 Xcode 開啟 `Runner.xcworkspace`。
2. 進行 Archive 並上傳至 TestFlight。
3. **注意**：確保 Provisioning Profile 已更新。

### 5.3 K8s 服務更新
1. 修改相關專案的 `deployment.yaml` 映像檔版本。
2. 執行 `kubectl apply -f deployment.yaml`。

---

## 6. 關鍵聯絡人 (Contact List)

| 角色 | 姓名 | 聯絡方式 | 負責事項 |
| :--- | :--- | :--- | :--- |
| 專案窗口 | Chris Chen | Redmine / Email | 需求確認、測試回饋 |
| 基礎設施 | [資訊部同事] | 內部分機 | 伺服器、VPN、K8s 權限 |
| 前開發者 | Kent Lin | [您的聯絡方式] | 架構諮詢、重大 Bug 背景說明 |

---
*嘉鈊科技 (Gamma Technology) - 內部機密文件*
