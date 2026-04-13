# Proxmox VE (PVE) 維運與設定手冊

本文件記錄嘉鈊科技地端虛擬化環境的配置邏輯與維護 SOP。

## 1. 硬體與節點配置
*   **物理主機**: [填入主機型號，如 Dell R740 / 自組伺服器]
*   **CPU/RAM 策略**: 
    *   預留 10-15% 資源給 Host OS 確保穩定。
    *   關鍵 VM 使用 `Host` CPU 類型以獲得最佳效能。
*   **儲存架構**:
    *   **local-zfs (ZFS)**: 存放系統鏡像與高效能磁碟需求。
    *   **NFS/NAS**: 存放備份檔與 ISO 安裝檔。

## 2. 網路與 VLAN 配置
*   **管理網段 (vmbr0)**: 固定 IP 位址，僅允許內部管理跳板機存取。
*   **業務網段 (VLAN 標籤)**:
    *   VLAN 10: Kubernetes Nodes
    *   VLAN 20: 測試環境
*   **MTU 設定**: 叢集內同步建議維持 1500，若有 10G 網卡需求可調整為 9000 (Jumbo Frames)。

## 3. VM/LXC 標準化 (Cloud-Init)
*   **範本製作**: 使用 Ubuntu 22.04 LTS Cloud-image。
*   **標準配置**: 2 vCPU, 4GB RAM, 32GB Disk, 預裝 `qemu-guest-agent`。
*   **自動化**: 透過 Cloud-init 注入 SSH Key，嚴禁使用明文密碼。

## 4. 備份與復原 (PBS)
*   **備份排程**: 每日凌晨 02:00 執行增量備份。
*   **保留政策**: 
    *   Keep Daily: 7
    *   Keep Weekly: 4
    *   Keep Monthly: 1
*   **復原測試**: 每季需執行一次關鍵 VM（如 K8s Master）的還原演練。

## 5. 常用指令
*   `qm status <vmid>`: 查看 VM 狀態。
*   `pvecm status`: 查看叢集健康狀況。
*   `zpool status`: 檢查磁碟陣列健康度。
