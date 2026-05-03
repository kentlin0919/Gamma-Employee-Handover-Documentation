# 02-infra-k8s-pve (基礎設施交接文件)

## 1. 專案概述
管理嘉鈊科技內部的基礎設施，包含 Proxmox VE (PVE) 虛擬化平台以及運行於其上的 Kubernetes (K8s) 集群。

## 2. 核心架構
- **虛擬化平台**: Proxmox VE (PVE)
- **容器編排**: K3s (Flannel VXLAN + 嵌入式 etcd)
- **儲存方案**: SMB CSI + local-path (⚠️ SMB 極不穩定，建議遷移)
- **負載均衡**: MetalLB (L2 Mode)

## 📖 核心維運文件 (必讀)

為了確保系統穩定，請務必依序閱讀並遵循以下手冊：

*   **[🛠️ 基礎設施維護手冊](./docs/K8S_OPERATIONS.md)**：包含每日/每週檢查 Checklist、證書管理與組件深度維護。
*   **[🔄 虛擬層重啟 SOP](./docs/RESTART_SOP.md)**：包含完整斷電恢復流程、PVE Web UI 操作與 SMB 凍結排除。
*   **[⚠️ 已知問題總整理](./docs/K3S_KNOWN_ISSUES.md)**：記錄了 SMB CSI、MetalLB 衝突等高風險問題的根因與解法。

## 🏗️ 基礎設施架構

本環境基於 **Proxmox VE** 虛擬化平台，運行單節點 **K3s** 叢集：

*   **PVE Host**: 10.0.5.10 (實體主機)
*   **K3s VM (gamma)**: 10.0.5.27 (運行 K3s Server)
*   **GitOps**: [ArgoCD](https://10.0.5.201) (負責服務自動化部署)
*   **反向代理**: [Caddy](http://10.0.5.202) (負責對外流量入口)
*   **儲存系統**: SMB CSI (網路磁碟) + local-path (本地磁碟)

### 3.3 內部系統維護
- **Redmine Dashboard**: 負責內部專案管理儀表板的開發與日常維護。

## 4. 資源分配
- **開發環境 (Staging)**: 運行於 Namespace `dev`。
- **正式環境 (Prod)**: 運行於 Namespace `prod`。

---
*嘉鈊科技 (Gamma Technology) - 內部機密文件*
