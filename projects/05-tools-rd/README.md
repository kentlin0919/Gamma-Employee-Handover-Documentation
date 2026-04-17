# 05-tools-rd (研發工具與自動化交接文件)

## 1. 專案概述
收集與開發嘉鈊科技內部研發團隊使用的輔助工具、自動化腳本及 CI/CD 優化工具。

## 2. 現有工具清單
- **Gemini CLI Config**: 自動化文件生成與程式碼分析設定。
- **Git Hooks**: 程式碼風格與 Commit 訊息檢查。
- **自動化部署腳本**: 針對 Brymen APP 的打包與分發腳本。
- **Go2RTC Streaming**: RTSP 轉 WebRTC 串流伺服器建置與 [Wowza 相容性說明](./docs/go2rtc-streaming-setup.md)。

## 3. 維護說明
- **Gemini CLI**: 設定檔位於本倉庫的 `.gemini/` 目錄。
- **自動化測試**: 整合於 CI 流程中的單元測試與 Linter。

---
*嘉鈊科技 (Gamma Technology) - 內部機密文件*
