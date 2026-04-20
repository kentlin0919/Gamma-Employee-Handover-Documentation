# Kubernetes Pod 掛載機敏檔案 (Secret) 操作手冊

## 任務背景
因應 ArgoCD 部署或其它需要私鑰 (`.pem`, `.zip`) 的情境，本文件說明如何將機敏檔案安全地掛載至 Kubernetes Pod 中。

## 1. 建立 Kubernetes Secret
首先需要將本機端的私鑰檔案轉換為 K8s 的 Secret 資源。

### 指令範例
假設檔案路徑為 `projects/02-infra-k8s-pve/argocd-gamma.2026-03-26.private-key.pem.zip`：

```bash
kubectl create secret generic argocd-private-key \
  --from-file=argocd-key.zip=projects/02-infra-k8s-pve/argocd-gamma.2026-03-26.private-key.pem.zip \
  -n <your-namespace>
```

*   **argocd-private-key**: Secret 的名稱。
*   **--from-file=key-name=path**: 將實體檔案對照到 Secret 內部的 key。

## 2. 建立 Pod 並掛載 Secret
在 Pod 的 YAML 配置中，透過 `volumes` 定義 Secret，並在 `volumeMounts` 中掛載到容器內之特定路徑。

### YAML 範例 (`pod.yaml`)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: argo-key-tester
  namespace: <your-namespace>
spec:
  containers:
  - name: tools
    image: alpine
    command: ["sleep", "3600"]
    volumeMounts:
    - name: key-volume
      mountPath: "/opt/keys" # 檔案會出現在容器內的此目錄
      readOnly: true
  volumes:
  - name: key-volume
    secret:
      secretName: argocd-private-key
```

### 部署指令
```bash
kubectl apply -f pod.yaml
```

## 3. 驗證檔案是否掛載成功

### A. 終端機驗證
進入 Pod 查看檔案：

```bash
kubectl exec -it argo-key-tester -n <your-namespace> -- ls /opt/keys
# 預期輸出: argocd-key.zip
```

### B. ArgoCD 介面驗證
如果您使用 ArgoCD 管理 Application，可以進入 **Resource Tree** 視圖進行視覺化確認：

![ArgoCD Resource Tree](https://10.0.5.201/applications/agentdvr?resource=)
> *註：截圖為 `agentdvr` 範例，可見 Pod 與 Secret (smb-creds) 的關聯。*

1.  **開啟應用詳情**：在 ArgoCD 點擊您的 Application。
2.  **查看 Resource Tree**：確認代表 `Secret` 的圖示與 `Pod` 圖示之間有連線（表示掛載關係已建立）。
3.  **檢查同步狀態**：確保頂部狀態顯示為 `Synced` 且 `Healthy`。

![ArgoCD Applications Overview](https://10.0.5.201/applications)

## 4. 安全注意事項 (Security)
*   **嚴禁 commit 機密資訊**: 絕對不要將產出的 Secret YAML 或對應的 `.zip` 檔案 commit 到公開 git repo。
*   **檔案清理**: 依照 [GEMINI.md](file:///Users/kent/project/Gamma-Employee-Handover-Documentation/GEMINI.md) 規範，一旦檔案已成功引入 K8s 且不再需要本機副本，請清理下載目錄。
*   **權限控制**: 建議給予 Pod 最小權限，並確保 `readOnly: true` 以防誤刪。

## 5. 相關參考
*   [K8S_OPERATIONS.md](file:///Users/kent/project/Gamma-Employee-Handover-Documentation/projects/02-infra-k8s-pve/docs/K8S_OPERATIONS.md)
*   [README.md](file:///Users/kent/project/Gamma-Employee-Handover-Documentation/projects/02-infra-k8s-pve/README.md)
