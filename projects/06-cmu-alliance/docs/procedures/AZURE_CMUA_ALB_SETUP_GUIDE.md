# Azure Application Gateway (ALB) 設定指南 (cmua-cmu 專屬資源集)

本文件提供 `cmua-cmu` 資源群組建立 Azure Application Gateway (ALB) 的可執行 SOP，適用於 `CMU Alliance` 專案目前的雙機應用架構。

本版已依 `2026-04-14` 透過 Context7 查核 Microsoft Learn 官方文件後整理，重點對齊下列要求：

* `Application Gateway v2` 需要使用 **專用子網路**，不可與其他工作負載共用。
* `Standard v2` 搭配公網前端時，應使用 **Standard / Static Public IP**。
* Application Gateway 子網路至少建議 `/27`，若考量 autoscaling 與後續維運，**建議 `/24`**。

## 1. 官方參考文件

1. Microsoft Learn: [Application Gateway infrastructure configuration](https://learn.microsoft.com/en-us/azure/application-gateway/configuration-infrastructure)
2. Microsoft Learn: [Quickstart - Direct web traffic with Azure Application Gateway - Azure portal](https://learn.microsoft.com/en-us/azure/application-gateway/quick-create-portal)

以下截圖已存入專案 `assets/`，可作為交接時的視覺參考：

![Microsoft Learn - Application Gateway infrastructure configuration](../../assets/cmua-appgw-config-infra.png)

![Microsoft Learn - Quickstart create application gateway in Azure portal](../../assets/cmua-appgw-quickstart-portal.png)

## 2. 本專案目標配置

| 項目 | 設定值 | 說明 |
| :--- | :--- | :--- |
| Subscription | 依現場實際訂閱 | 建議先確認是否為 CMUA 正式使用中的訂閱 |
| Resource Group | `cmua-cmu` | CMUA Azure 資源群組 |
| Region | `Japan East` | 必須與既有網路與 VM 同區 |
| Application Gateway 名稱 | `cmua-cmu-alb` | 專案標準命名 |
| SKU / Tier | `Standard_v2` | 目前以 ALB 為目標，不含 WAF |
| Virtual Network | `cmua-cmu-Network` | 既有 VNet |
| ALB Subnet | `appgw-subnet` | ALB 專用子網路 |
| Frontend Public IP | `cmua-cmu-frontend-ip` | `Standard` + `Static` |
| Backend Pool | `cmua-cmu-ap-pool` | 目標 VM: `cmua-cmu-ap1`、`cmua-cmu-ap2` |
| Backend Setting | `cmua-cmu-ap-setting` | 預設 HTTP，實際埠號依服務確認 |
| Listener | `cmua-cmu-http-listener` | 初版先用 HTTP/80 |
| Routing Rule | `cmua-cmu-routing-rule` | Basic rule |

## 3. 前置確認

建立前先完成以下檢查，否則容易在建立流程中卡住：

* [ ] 已確認操作帳號對 `cmua-cmu` 具備至少 `Network Contributor` 或更高權限
* [ ] `cmua-cmu-ap1`、`cmua-cmu-ap2` 皆已存在且狀態正常
* [ ] `cmua-cmu-Network` 與兩台 VM 位於同一個 `Japan East`
* [ ] 已確認應用實際對外服務埠號，例如 `80`、`8080` 或自訂 port
* [ ] 已確認健康檢查路徑，例如 `/health`、`/` 或應用可回 `200` 的 endpoint
* [ ] 已確認 NSG 沒有阻擋來自 Application Gateway 子網到後端 VM 的必要流量

## 4. 建立 ALB 專用子網路

根據 Microsoft Learn，Application Gateway v2 必須使用專用子網路；該子網路中不應放置其他 VM、Private Endpoint 或一般工作負載。

### 4.1 檢查既有子網路

1. 進入 Azure Portal。
2. 搜尋並開啟 `cmua-cmu-Network`。
3. 進入左側 `Subnets`。
4. 確認是否已存在 `appgw-subnet`。

### 4.2 建立子網路

若尚未建立，請新增：

| 欄位 | 建議值 |
| :--- | :--- |
| Name | `appgw-subnet` |
| Address range | 優先使用 `/24`，例如 `10.0.1.0/24` |
| Network security group | 先沿用現場政策，避免自行加上過度限縮規則 |
| Route table / NAT gateway / service endpoint | 若無明確需求，先保持預設 |

### 4.3 子網路規劃原則

* 最低至少保留 `/27`
* 若考量 `Standard_v2` autoscaling 與 hitless maintenance，建議直接保留 `/24`
* 不可與 `BackendSubnet` 或資料庫 Private Endpoint 所在子網路重疊

## 5. 建立 Public IP

1. 於 Azure Portal 搜尋 `Public IP addresses`
2. 點選 `Create`
3. 填入下列值：

| 欄位 | 設定值 |
| :--- | :--- |
| Resource group | `cmua-cmu` |
| Name | `cmua-cmu-frontend-ip` |
| Region | `Japan East` |
| SKU | `Standard` |
| Assignment | `Static` |
| Availability zone | 依現場策略，若無特別要求可先維持預設 |

4. 完成後建立

## 6. 建立 Application Gateway

1. 搜尋 `Application gateways`
2. 點選 `Create`
3. 於 `Basics` 分頁填入：

| 欄位 | 設定值 |
| :--- | :--- |
| Subscription | 現場使用中的 CMUA 訂閱 |
| Resource group | `cmua-cmu` |
| Application gateway name | `cmua-cmu-alb` |
| Region | `Japan East` |
| Tier | `Standard V2` |
| Enable autoscaling | 建議開啟，Min instance 依現場容量需求 |
| Virtual network | `cmua-cmu-Network` |
| Subnet | `appgw-subnet` |
| Frontend IP type | `Public` |
| Public IP address | `cmua-cmu-frontend-ip` |

4. 進入 `Frontends` / `Configuration` 相關頁面時，確認 Portal 沒有自動替換掉既有命名
5. 完成建立

## 7. 建立 Backend Pool

Application Gateway 建立完成後，進入 `Backend pools`：

| 欄位 | 設定值 |
| :--- | :--- |
| Pool name | `cmua-cmu-ap-pool` |
| Target type | `Virtual machine` |
| Targets | `cmua-cmu-ap1`、`cmua-cmu-ap2` |

操作原則：

* 若 Portal 可直接選 VM，優先使用 VM 綁定
* 若現場改為 NIC / IP 方式管理，也可用 private IP 加入，但需同步記錄在本文件

## 8. 建立 Backend Setting 與 Health Probe

進入 `Backend settings`，新增：

| 欄位 | 設定值 |
| :--- | :--- |
| Name | `cmua-cmu-ap-setting` |
| Backend protocol | `HTTP` |
| Backend port | 依實際服務埠號，常見 `80` |
| Cookie-based affinity | 依系統是否依賴 session sticky 決定，若未確認先 `Disabled` |
| Connection draining | 視現場需求，可先關閉 |
| Request timeout | `20` 秒起 |
| Override with new host name | 若後端不是 host-based routing，通常不需開啟 |

Health Probe 建議同步建立，不要只依賴預設探查：

| 欄位 | 建議值 |
| :--- | :--- |
| Probe name | `cmua-cmu-ap-probe` |
| Protocol | `HTTP` |
| Host | 依實際服務需求，若無特殊 host header 可留空或使用 `127.0.0.1` 對應模式 |
| Path | 優先使用 `/health`，若應用沒有則改為能穩定回 `200` 的路徑 |
| Interval | `30` 秒 |
| Timeout | `30` 秒 |
| Unhealthy threshold | `3` |

實務要求：

* Probe 路徑必須能穩定回應 `200-399`
* 若站台會對未登入使用者轉址，請勿直接拿登入頁作為 probe 路徑
* 若後端使用 IIS / Kestrel / Nginx 反向代理，請先在 VM 上本機測過 probe path

## 9. 建立 Listener

進入 `Listeners`，新增：

| 欄位 | 設定值 |
| :--- | :--- |
| Name | `cmua-cmu-http-listener` |
| Frontend IP | `cmua-cmu-frontend-ip` |
| Protocol | `HTTP` |
| Port | `80` |
| Listener type | `Basic` |

備註：

* 若要直接上正式環境，建議同步規劃 `HTTPS / 443`
* 若後續掛憑證，請另建 `HTTPS listener`，不要直接覆蓋原設定後卻不記錄

## 10. 建立 Routing Rule

進入 `Rules`，新增 Basic rule：

| 欄位 | 設定值 |
| :--- | :--- |
| Rule name | `cmua-cmu-routing-rule` |
| Priority | `100` 或依現場規劃 |
| Listener | `cmua-cmu-http-listener` |
| Backend target | `cmua-cmu-ap-pool` |
| Backend settings | `cmua-cmu-ap-setting` |

目前 `cmua-cmu` 先以單一站台導流為主，因此使用 `Basic` rule 即可。若未來需要：

* API / Web 拆流
* 多站台 host-based routing
* HTTP to HTTPS redirect

則再改用多 listener 與 path-based / redirect 規則。

## 11. 驗證流程

建立完成後，請至少完成以下驗證：

### 11.1 Azure Portal 驗證

* `Overview` 顯示 `Running`
* `Backend health` 看到 `cmua-cmu-ap1`、`cmua-cmu-ap2` 為 `Healthy`
* `Frontend public IP` 已綁定 `cmua-cmu-frontend-ip`
* `Listener`、`Backend setting`、`Rule` 命名均符合本 SOP

### 11.2 應用驗證

1. 從公司可測網路或受控測試端對 Public IP 發送 HTTP 請求
2. 驗證首頁可正常回應
3. 若有健康檢查頁，直接測 probe path 是否為 `200`
4. 停用其中一台 VM 的網站服務，確認 ALB 會將不健康節點排除

### 11.3 VM 側驗證

在 `cmua-cmu-ap1`、`cmua-cmu-ap2` 上確認：

* 本機服務有在預期 port 上 listen
* Windows Firewall / NSG 沒有擋掉來自 `appgw-subnet` 的流量
* 若應用依賴 SQL，仍可私網連到 `cmua-cmu-db`

### 11.4 Backend VM 防火牆配置 (Windows Firewall)

兩台 VM (`10.0.0.4`, `10.0.0.5`) 必須開放 60011 與 60001 埠號。請以系統管理員權限開啟 PowerShell 執行：

```powershell
# 開放 Inbound (輸入)
New-NetFirewallRule -DisplayName "Allow 60011 Inbound" -Direction Inbound -Protocol TCP -LocalPort 60011 -Action Allow
New-NetFirewallRule -DisplayName "Allow 60001 Inbound" -Direction Inbound -Protocol TCP -LocalPort 60001 -Action Allow

# 開放 Outbound (輸出)
New-NetFirewallRule -DisplayName "Allow 60011 Outbound" -Direction Outbound -Protocol TCP -LocalPort 60011 -Action Allow
New-NetFirewallRule -DisplayName "Allow 60001 Outbound" -Direction Outbound -Protocol TCP -LocalPort 60001 -Action Allow

# 驗證規則
Get-NetFirewallRule | Where-Object { $_.DisplayName -like "Allow 6000*" }
```

### 11.5 NSG 規則修正步驟 (2026-04-14)

根據現有診斷結果（ap2 缺少 Outbound 規則等），應執行以下修正步驟：

1.  **ap1-nsg — 刪除誤加規則**: 刪除 `AllowGatewayManagerInbound` (Priority 330, 65200-65535)，使用者無此需求。
2.  **ap1-nsg — 新增 Inbound**: 名稱為 `cmua-cmu-api-input` | Port: `60001` | Priority: `330`。
3.  **ap1-nsg — 新增 Outbound**: 名稱為 `cmua-cmu-api-output` | Port: `60001` | Priority: `340`。
4.  **ap2-nsg — 新增 Inbound**: 名稱為 `cmua-cmu-api-input` | Port: `60001` | Priority: `320`。
5.  **ap2-nsg — 新續 Outbound (60011)**: 名稱為 `cmua-cmu-web-output` | Port: `60011` | Priority: `330`。
6.  **ap2-nsg — 新增 Outbound (60001)**: 名稱為 `cmua-cmu-api-output` | Port: `60001` | Priority: `340`。

### 最終 NSG 規則總表 (驗證基準)

| NSG | 方向 | 規則名稱 | Port | 狀態 |
| :--- | :--- | :--- | :--- | :--- |
| ap1-nsg | Inbound | `cmua-cmu-web-input` | 60011 | 原本已有 |
| ap1-nsg | Inbound | `cmua-cmu-api-input` | 60001 | ✅ 已新增 |
| ap1-nsg | Outbound | `cmua-cmu-web-output` | 60011 | 原本已有 |
| ap1-nsg | Outbound | `cmua-cmu-api-output` | 60001 | ✅ 已新增 |
| ap2-nsg | Inbound | `cmua-cmu-web-input` | 60011 | 原本已有 |
| ap2-nsg | Inbound | `cmua-cmu-api-input` | 60001 | ✅ 已新增 |
| ap2-nsg | Outbound | `cmua-cmu-web-output` | 60011 | ✅ 已新增 |
| ap2-nsg | Outbound | `cmua-cmu-api-output` | 60001 | ✅ 已新增 |

## 12. 常見失敗點

### 12.1 無法建立 Application Gateway

常見原因：

* `appgw-subnet` 不存在
* 子網路與其他子網重疊
* 子網路內已有不相容資源
* Region 與 VNet / Public IP 不一致

### 12.2 Backend 顯示 Unhealthy

常見原因：

* **NSG 擋住監控流量 (Skv v2)**：必須在 NSG 開放來源 `GatewayManager` 存取目的 Port `65200-65535`。
* **API 回傳 401**：AGW 預設接受 200-399。若 API 需要認證，請建立自訂 Health Probe 指向不需認證的路徑（如 `/health` 或 `/ping`）。
* **防火牆未開放**：未執行 11.4 節之 PowerShell 腳本。
* 後端 port 寫錯、Probe path 寫錯、VM 本身服務未啟動。

### 12.3 前端可連到 Public IP，但網站無法開啟

常見原因：

* **API Pool 未關聯規則**：確認 Rule 是否已配置 Path-based routing（例如 `/api/*` 指向 `api-pool`）。
* Listener 與 Rule 沒綁定成功。
* Backend setting 沒有指向正確的 backend pool。

## 13. 上線後建議補強

目前本 SOP 先滿足 `cmua-cmu` 可用的 ALB 最小配置，正式上線前建議再補：

1. 建立 `HTTPS listener (443)` 並掛入正式憑證
2. 規劃 `HTTP -> HTTPS redirect`
3. 若網站對外，評估升級為 `WAF_v2`
4. 將 Probe path 改為專用健康檢查端點，而非首頁
5. 建立 Azure Monitor / Alert 規則監控 Backend health 與 5xx

## 14. 交接紀錄

* 文件更新日期：`2026-04-14`
* 本次補強內容：整合 2026-04-14 之深度診斷報告、NSG 修正步驟與防火牆 PowerShell 腳本。

## 15. 待處理項目 (Pending Items)

1. **Application Gateway Routing Rule**: `api` pool 目前無導流規則，需確認外部存取之路徑 (Path) 或端口 (Port)，以建立對應的 Listener 與 Rule。
2. **Custom Health Probe for 60001**: 目前探查路徑會因 401 被拒絕，需在後端新增一個不需認證的 Endpoint (如 `/health`) 並更新 AGW Probe 設定。

---
*嘉鈊科技 (Gamma Technology) - 內部機密文件*
