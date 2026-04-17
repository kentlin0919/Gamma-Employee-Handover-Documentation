# Go2RTC 串流伺服器部署與 Wowza 相容性設定

---

## 1. 專案概述 (Project Overview)
- **專案名稱**：Go2RTC RTSP 串流轉換服務
- **專案目標**：將攝影機的原生 RTSP 串流轉換為廣泛支援的 WebRTC 及 HTTP-MSE 格式，提供低延遲的網頁監看方案。
- **關鍵功能**：
  - RTSP -> WebRTC (Low Latency)
  - 支援 Wowza Cloud 特殊格式處理

## 2. 技術架構 (Technical Architecture)
### 2.1 技術棧 (Tech Stack)
- **語言/框架**：Go (Go2RTC 核心)
- **多媒體核心**：FFmpeg (用於處理非標準 RTSP 交涉)
- **基礎設施**：Docker / OrbStack (Mac 環境)

### 2.2 配置檔案位置
- `docker-compose.yml`：定義服務容器、通訊埠映射與掛載路徑。
- `go2rtc.yaml`：定義串流來源、WebRTC 參數與日誌層級。

## 3. 部署方式 (Getting Started)
### 3.1 啟動指令
切換至專案目錄並執行：
```bash
docker compose up -d
```

### 3.2 關鍵配置 (go2rtc.yaml)
針對 **Wowza Cloud** 或嚴格的 RTSP 伺服器，必須使用 `ffmpeg:` 前綴：
```yaml
streams:
  # 使用 ffmpeg: 前綴以提高相容性
  cam1: ffmpeg:rtsp://716f898c7b71.entrypoint.cloud.wowza.com:1935/app-8F9K44lJ/304679fe_stream2#video=copy#audio=copy
```

## 4. 關鍵運作與維護 (Maintenance)
- **日誌檢查**：`docker compose logs go2rtc -f`
- **服務介面**：部署後可透過 `http://localhost:1984` 進入儀表板查看串流狀態。

## 5. 已知問題與解決方案 (Known Issues)
### 5.1 RTSP SETUP 階段失敗
- **徵兆**：日誌顯示 `wrong response on SETUP` 隨後出現 `EOF`。
- **原因**：原生 go2rtc RTSP 客戶端在與部分 Wowza 版本交涉時，無法正確處理特定的 Interleaved mode 或 User-Agent 要求。
- **解決方案**：
  - **強烈建議使用 FFmpeg 作為中介**：在串流 URL 前加上 `ffmpeg:`。
  - **效能優化**：串流後方加上 `#video=copy`，確保僅轉換封裝而不消耗 CPU 進行轉碼。

---
*嘉鈊科技 (Gamma Technology) - 內部機密文件*
