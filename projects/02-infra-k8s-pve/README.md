# 02-infra-k8s-pve (基礎設施交接文件)

## 1. 專案概述
管理嘉鈊科技內部的基礎設施，包含 Proxmox VE (PVE) 虛擬化平台以及運行於其上的 Kubernetes (K8s) 集群。

## 2. 核心架構
- **虛擬化平台**: Proxmox VE (PVE)
- **容器編排**: Kubernetes (K8s)
- **存儲方案**: [如：NFS / Ceph]
- **網路配置**: [如：Calico / MetalLB]

## 3. 關鍵管理操作
### 3.1 PVE 管理
- **節點資訊**: [列出主要 Node IP]
- **備份策略**: 每日 VM 快照與備份排程。
- **認證方式**: 透過 PVE Web UI (需 VPN 權限)。

### 3.2 K8s 管理
- **叢集狀態**: `kubectl get nodes`
- **服務部署**: 使用 Helm Chart 或 YAML 定義。
- **特定服務遷移**: 已完成 Caddy (Linux) 遷移並部署至 K8s。
- **監控系統**: Prometheus + Grafana (URL: [填寫監控連結])。

### 3.3 內部系統維護
- **Redmine Dashboard**: 負責內部專案管理儀表板的開發與日常維護。

## 4. 資源分配
- **開發環境 (Staging)**: 運行於 Namespace `dev`。
- **正式環境 (Prod)**: 運行於 Namespace `prod`。

---
*嘉鈊科技 (Gamma Technology) - 內部機密文件*
