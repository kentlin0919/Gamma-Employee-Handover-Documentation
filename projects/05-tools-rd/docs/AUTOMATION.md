# 研發自動化與 CI/CD 工具說明 (AUTOMATION.md)

## 1. Gemini CLI 工具
- **用途**: 自動化程式碼分析、Bug 根因定位、文件生成。
- **設定**: 參考根目錄 `.gemini/settings.json` 與 `GEMINI.md`。

## 2. 自動化部署腳本 (CI/CD)
- **Android**: `fastlane` 腳本自動上傳 APK 至 internal track。
- **iOS**: 透過 `xcodebuild` 指令結合內部分發工具進行打包。

## 3. Git Hooks
- 包含 Pre-commit 檢查：
  - Linter 檢查。
  - Commit Message 格式檢查（需包含 #IssueID）。
  - 嚴禁提交 `.env` 等敏感檔案。
