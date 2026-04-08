# 藍牙通訊協定與封包解析 (BLE_PROTOCOL.md)

## 1. 封包格式
電表回傳之封包為定長 Byte 陣列，包含：
- **Header**: 封包起始標記。
- **Payload**: 包含數值、小數點位、單位前綴 (Prefix)、功能符號。
- **Checksum**: 異或校驗 (XOR)。

## 2. 數據解碼邏輯
- **Checksum 檢查**: 若校驗失敗，該筆資料應視為 `null` 處理，避免圖表出現跳變 (#9312)。
- **單位換算**: 需將 Payload 轉換為實際讀值，並套用 `metricPrefix` (如 k, M, m, μ)。

## 3. 控制指令 (Command Set)
- `0x01`: 設定 RTC 時間。
- `0x05`: Data Logger 模式切換。
- `0x08`: 讀取/設定裝置輸出頻率 (Output Rate)。
