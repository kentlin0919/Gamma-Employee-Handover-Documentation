# Caddy 反向代理與自動 HTTPS 設定手冊

本文件說明如何使用 Caddy 進行網域管理與內部服務轉發，包含 SSL 證書自動化設定。

## 1. 設定管理 (Caddyfile)
*   **路徑**: `/etc/caddy/Caddyfile`
*   **重載配置**: `caddy reload --config /etc/caddy/Caddyfile` 或 `systemctl reload caddy`

> [!IMPORTANT]
> **Caddy 官方強烈建議**: 修改配置後永遠使用 `caddy reload`，**不要停止再啟動 Caddy**。
> `caddy reload` 透過 API 優雅地套用新配置，不會中斷任何經有連線 (零停機重載)。
> 若使用 `systemctl restart caddy` 則會造成短暫服務中斷。
> — [來源: Caddy Getting Started](https://caddyserver.com/docs/getting-started)

## 2. 反向代理範例
```caddy
example.com {
    reverse_proxy 192.168.10.10:8080 {
        header_up Host {host}
        header_up X-Real-IP {remote_host}
    }
}
```

## 3. SSL 證書管理 (ACME)
*   **自動 HTTPS**: 預設使用 Let's Encrypt / ZeroSSL 進行 HTTP-01 驗證。
*   **DNS-01 驗證 (適用於內部網域或 Azure)**:
    *   若使用 Azure DNS，需安裝 `caddy-dns/azure` 插件。
    *   需設定 `AZURE_TENANT_ID`, `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET` 環境變數。

## 4. 監控與日誌
*   **日誌路徑**: `/var/log/caddy/access.log`
*   **狀態檢查**: `systemctl status caddy`
*   **即時查看日誌**: `journalctl -u caddy -f`

## 5. 安全強化
*   **HSTS**: 預設啟用。
*   **加密協議**: 強制使用 TLS 1.2 或 1.3。
*   **權限限制**: Caddy 以非 root 使用者執行，確保系統安全。

## 6. 維運指令快速參考

```bash
# 零停機重載配置 (推薦)
caddy reload --config /etc/caddy/Caddyfile

# 透過 systemd 重載 (等效於上面的指令)
sudo systemctl reload caddy

# 查看服務狀態
sudo systemctl status caddy

# 即時查看日誌
journalctl -u caddy -f

# 驗證配置檔語法 (不套用)
caddy validate --config /etc/caddy/Caddyfile

# 若使用 JSON 配置 (SystemD override)
# ExecStart=/usr/bin/caddy run --environ --config /etc/caddy/caddy.json
# ExecReload=/usr/bin/caddy reload --config /etc/caddy/caddy.json
```

> [!TIP]
> **`caddy reload` 參數說明**:
> - `--config <path>`: 指定配置檔路徑
> - `--adapter <name>`: 指定配置轉接器 (非 JSON 格式時使用)
> - `--force`: 即使配置未變也強制重載
> — [來源: Caddy Command Line](https://caddyserver.com/docs/command-line)
