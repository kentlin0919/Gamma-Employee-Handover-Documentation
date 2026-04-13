# Kubernetes (K8s) 叢集操作維運手冊

本文件提供嘉鈊科技 K8s 叢集的管理、維護與故障排除 SOP。

## 1. 叢集架構
*   **部署工具**: [Kubeadm / RKE2 / k3s]
*   **網路外掛 (CNI)**: [Calico / Flannel / Cilium]
*   **入口控制器 (Ingress)**: [Nginx Ingress / Caddy Ingress]
*   **儲存外掛 (CSI)**: [Local Path / Longhorn / Azure Disk]

## 2. 節點管理
*   **查看節點狀態**: `kubectl get nodes -o wide`
*   **排除/恢復節點**: 
    *   `kubectl drain <node-name> --ignore-daemonsets`
    *   `kubectl uncordon <node-name>`
*   **憑證更新**: 使用 `kubeadm certs renew all` (若使用 Kubeadm)。

## 3. 工作負載操作
*   **快速部署**: `kubectl apply -f <yaml-file>`
*   **動態擴展**: `kubectl scale deployment <name> --replicas=3`
*   **滾動更新 (Rollout)**:
    *   `kubectl rollout status deployment <name>`
    *   `kubectl rollout undo deployment <name>` (復原版本)

## 4. 監控與日誌分析
*   **Pod 日誌**: `kubectl logs -f <pod-name> -n <namespace>`
*   **資源使用度**: `kubectl top nodes` / `kubectl top pods -A`
*   **異常診斷**: `kubectl describe pod <pod-name> -n <namespace>`

## 5. 常見故障排除 (Troubleshooting)
*   **Pending 狀態**: 檢查 `describe` 中的 Events，通常是資源不足或 `StorageClass` 問題。
*   **CrashLoopBackOff**: 檢查應用程式日誌 (`kubectl logs`) 與 `env` 設定。
*   **DNS 解析失敗**: 檢查 `coredns` Pod 狀態與 Service IP 路由。

## 6. 憑證與機密管理 (Secrets)
*   **嚴禁提交 Secrets YAML 到 Git**。
*   **存取機密**: `kubectl get secret <name> -o yaml`
*   **建議工具**: 使用 `Sealed Secrets` 或與 `Azure Key Vault` 整合。
