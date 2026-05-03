# 🛠️ Gamma 基礎設施維護手冊 (Maintenance Manual)

本手冊為嘉鈊科技 K3s 基礎設施的長期維運指南，包含例行健康檢查、深度組件維護及緊急故障處理流程。

---

## 📋 一、例行健康檢查 Checklist (建議每週執行)

| 檢查項目 | 指令 | 正常標準 |
| :--- | :--- | :--- |
| **節點健康** | `kubectl get nodes` | 所有節點狀態為 `Ready` |
| **Pod 穩定性** | `kubectl get pods -A | grep -v Running | grep -v Completed` | 輸出應為空 (無 Error/CrashLoop) |
| **磁碟空間** | `df -h` (在 PVE Node 上) | 使用率 < 85% |
| **K3s 服務** | `sudo systemctl status k3s` | 顯示 `active (running)` |
| **證書效期** | `sudo k3s certificate check` | 剩餘天數 > 30 天 |

---

## ⚙️ 二、K3s 核心組件維護

### 1. 服務管理與重啟
K3s 服務重啟時會觸發 Pod 重建（受 containerd 版本影響），請謹慎操作。
*   **安全重啟流程**:
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl restart k3s
    ```
*   **查看即時日誌**:
    ```bash
    # 追蹤 K3s 系統錯誤
    sudo journalctl -u k3s -f | grep -i "error"
    ```

### 2. 證書管理 (Certificate Management)
K3s 證書預設有效期為 1 年。
*   **檢查證書**: `sudo k3s certificate check`
*   **手動輪轉證書**:
    ```bash
    # 如果證書即將過期，執行此指令後重啟 k3s
    sudo k3s certificate rotate
    sudo systemctl restart k3s
    ```

---

## 🌐 三、網路與負載平衡維護 (MetalLB / Caddy)

### 1. MetalLB 狀態確認
如果外部無法連線至服務，請檢查 MetalLB Speaker 是否正常廣播。
*   **查看 Speaker 日誌**:
    ```bash
    kubectl logs -l component=speaker -n metallb-system --tail=50
    ```
*   **關鍵指標**: 搜尋日誌中是否有 `announcing from node`，這代表節點正在對外宣告 LoadBalancer IP。

### 2. Caddy 反向代理維護
Caddy 作為流量入口，必須維持「零停機」配置更動。
*   **驗證配置 (不套用)**:
    ```bash
    # 在 Caddy 容器內或安裝有 caddy 的主機執行
    caddy validate --config /etc/caddy/Caddyfile
    ```
*   **零停機熱載入 (Hot Reload)**:
    ```bash
    # 嚴禁使用 restart，請使用 reload
    caddy reload --config /etc/caddy/Caddyfile
    ```

---

## 📦 四、GitOps 與 CD 維護 (ArgoCD)

### 1. 同步卡住 (Out of Sync) 處置
當 ArgoCD 顯示 `ComparisonError` 或長時間 `Syncing`：
*   **強制重整**: 在 UI 點擊 `Refresh` -> `Hard Refresh`。
*   **重啟 Repo Server**:
    ```bash
    kubectl rollout restart deployment argocd-repo-server -n argocd
    ```

### 2. 資料庫維護
ArgoCD 會在 Redis 快取大量資料，若 UI 異常緩慢：
*   **清理 Redis**: `kubectl delete pod -l app.kubernetes.io/name=argocd-redis -n argocd` (Pod 重建後會自動重建快取)。

---

## 💾 五、儲存空間與 SMB CSI 專項處置

### 1. 處理「殭屍掛載」導致的 Node Unknown
這是本叢集最高頻率發生的嚴重問題。當 SMB 伺服器斷線，Kubelet 會因等待 I/O 而凍結。
*   **診斷**: 
    - 執行 `df -h` 卡住。
    - 執行 `ps aux | grep k3s` 看到大量 `D` 狀態 (Uninterruptible sleep) 進程。
*   **強制修復 (必須在 PVE Console 執行)**:
    ```bash
    # 1. 找出掛載點
    mount | grep cifs
    
    # 2. 強制解除掛載 (即便裝置忙碌中)
    sudo umount -f -l /var/lib/kubelet/pods/<Pod-UID>/volumes/kubernetes.io~csi/pvc-<ID>/mount
    ```

### 2. 磁碟清理指令
*   **清理未使用的容器鏡像**:
    ```bash
    sudo k3s crictl rmi --prune
    ```
*   **清理系統日誌 (Ubuntu)**:
    ```bash
    sudo journalctl --vacuum-time=7d
    ```

---

## 🚨 六、緊急災難恢復 (Panic Button)

若叢集完全失去響應，請依序執行：
1.  **進入 PVE 控制台**，強制重啟虛擬機 `gamma`。
2.  **在啟動完成前**，檢查實體機與 SMB 伺服器的網路連線。
3.  **登入系統後**，立即執行 `sudo systemctl status k3s`。
4.  若服務啟動失敗，檢查 `/var/lib/rancher/k3s/server/db/etcd` 是否損壞。

---

## 🔗 相關資源
*   [重啟 SOP (RESTART_SOP.md)](./RESTART_SOP.md)
*   [已知問題總整理 (K3S_KNOWN_ISSUES.md)](./K3S_KNOWN_ISSUES.md)
