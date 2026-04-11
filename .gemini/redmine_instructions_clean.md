# Redmine MCP 使用準則 (Redmine Instructions)

本文件規範 Gemini CLI 在處理「嘉鈊科技 (Gamma)」相關專案時，如何有效利用 Redmine MCP (Model Context Protocol) 進行任務追蹤、分析與文件化。

## 1. 核心專案資訊
- **主要專案 ID**: `75` (GX-BRYMEN APP 第二階段)
- **主要負責人**: Kent Lin (接手/維護者)
- **Redmine URL**: `http://redmine.example.com` (請根據實際環境調整)

## 2. 工具目錄 (Tool Catalog)
Gemini CLI 具備以下 Redmine MCP 工具，應根據任務階段選擇合適工具：
- `mcp_redmine_redmine_paths_list`: **中繼資料探索**。查詢 Redmine API 支援的所有路徑。
- `mcp_redmine_redmine_paths_info`: **路徑詳解**。取得特定 API 路徑的參數格式與規格 (OpenAPI)。
- `mcp_redmine_redmine_request`: **核心操作**。執行 GET (讀取)、POST (新增)、PUT (更新) 等請求。
- `mcp_redmine_redmine_upload`: **附件上傳**。取得 token 以將檔案關聯至 Issue。
- `mcp_redmine_redmine_download`: **附件下載**。讀取 Issue 附件 (如截圖、日誌、設計稿) 以進行深度分析。

## 3. 任務分析流程 (Research Phase)
在執行程式碼修改或撰寫文件前，必須完成以下步驟：
- **讀取 Issue 與日誌**: 優先使用 `mcp_redmine_redmine_request` 讀取 `journals`，追蹤討論歷史。
  - `path`: `/issues/{issue_id}.json?include=journals,attachments,relations`
- **中繼資料查詢**: 若需更新狀態或分類，先確認系統支援的清單。
  - 查詢議題狀態: `/issue_statuses.json`
  - 查詢優先等級: `/enumerations/issue_priorities.json`
- **深度研究**: 若 Issue 附有相關文件或圖片，使用 `mcp_redmine_redmine_download` 下載分析。
- **技術債識別**: 追蹤標註為 `tracker: "Bugs"` 或描述包含 "Bug", "Refactor", "Hack" 的任務。

## 4. 文件與任務串接 (Documentation Linking)
所有產出的 Markdown 文件 (如 `docs/ARCHITECTURE.md` 或 `README.md`) 必須落實：
- **超連結格式**: `[#IssueID](http://redmine.example.com/issues/IssueID)`。
- **上下文關聯**: 描述模組問題時必須標註對應 Issue ID。範例：
  > *初始化邏輯目前存在指令衝突問題，詳見 [#11711](...)*。

## 5. 任務狀態更新 (Execution Phase)
任務完成或取得進度後：
- **結構化更新**: 更新描述或加入 `notes` 時，需包含：
  1. 修復方案/開發內容簡述。
  2. 受影響的檔案路徑。
  3. 驗證方法或測試結果。
- **上傳證明**: 若產生新架構圖 (Mermaid) 或測試截圖，先 `upload` 取得 `token`，再透過 `PUT` 關聯。

## 6. 安全與隱私規範 (Security & Risk)
- **嚴禁洩漏**: 絕對禁止在 Redmine 中紀錄 `.env`、API Key、私人密鑰或資料庫密碼。
- **寫入確認**: 所有 `POST/PUT/DELETE` 操作必須符合當前分配的任務權限，嚴禁修改非相關任務。
- **敏感資訊遮罩**: 在回報測試結果時，應遮罩個人手機號碼或不必要的系統絕對路徑。

## 7. 常用 API 路徑參考
- **列出專案所有議題**: `/issues.json?project_id=75&status_id=open`
- **查詢特定類別 (如 Bug)**: `/issues.json?project_id=75&tracker_id=1`
- **查詢專案資訊**: `/projects/75.json?include=trackers,issue_categories`
- **更新任務狀態**: `PUT /issues/{issue_id}.json` (Body: `{"issue": {"status_id": 3, "notes": "Done"}}`)
