# Brymen APP 系統架構與狀態管理 (ARCHITECTURE.md)

## 1. 核心架構
本 App 採用 **GetX** 作為狀態管理與依賴注入框架，確保 UI 與業務邏輯分離。

### 目錄職責：
- `lib/controllers/`: 存放業務邏輯，如 `BleController`, `RecordController`。
- `lib/services/`: 藍牙底層通訊與實體層 (Physical Layer) 處理。
- `lib/models/`: 定義電表回傳之資料格式與 Hive Schema。
- `lib/isolates/`: 處理大檔案 CSV 解析，避免阻塞 UI 執行緒。

## 2. 藍牙初始化狀態機
連線後的初始化流程如下 (Ref: #11711)：
1. **Connect**: 建立 BLE 連結。
2. **Discover Services**: 搜尋特徵值。
3. **Sync Time**: 發送 `SetRTC` 指令。
4. **Get Configuration**: 讀取電表當前設定（注意：BM109x 不支援 `GetBleDataOutputInterval`，需在此處做型號判斷）。
5. **Start Notify**: 開始接收數據封包。

## 3. 即時圖表平移模型 (xVal Delta Points)
為了支撐長時間記錄的平移效果，我們引入了 `xVal` 累加器：
- **原理**: 當新點位進入時，不重新繪製整張圖，而是更新 X 軸位移量。
- **Tooltip 互動**: 觸控點需扣除當前 `xVal` 偏移，方能正確對應到 `commandData` 的索引。
