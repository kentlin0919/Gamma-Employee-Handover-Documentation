# Azure SFTP 服務端到端建立指南 (Windows VM + Azure File Share)

本指南提供從 Azure Portal 資源建立、Windows VM 設定到 WinSCP 連線測試的完整詳細步驟。為了確保服務穩定性與安全性，本手冊統一採用**「SYSTEM 帳戶認證 + 符號連結」**之標準實作方式。

---

## Step 1：建立 Azure Storage Account

1.  **進入 Portal**：上方搜尋欄輸入「**儲存體帳戶**」→ 點擊「建立」。
2.  **基本設定**：
    - **資源群組**：選擇 `cmua-cmu`。
    - **儲存體帳戶名稱**：輸入唯一名稱（如 `cmuacmustorage`）。
    - **區域**：`Japan East`。
    - **效能**：標準 (Standard)。
    - **備援**：LRS (本地備援儲存體，成本效益最高)。
3.  **檢閱 + 建立**：點擊「建立」並等待部署完成（約 30 秒）。

---

## Step 2：建立 Azure File Share

1.  進入剛建好的 Storage Account → 左側選單「**資料儲存**」→「**檔案共用**」。
2.  點擊「**+ 檔案共用**」。
3.  **設定**：
    - **名稱**：`appfiles`。
    - **存取層**：交易最佳化。
    - **配額**：依需求設定（例如 `100` GB）。
4.  點擊「建立」。

---

## Step 3：VM 儲存體認證與連線設定

為了讓 OpenSSH 服務（以 SYSTEM 帳號執行）能持續存取網路分享，必須將認證存入 Windows 憑證管理員。

1.  **取得金鑰**：進入 `appfiles` 檔案共用 → 點擊「**連線**」→ 選擇「**Windows**」分頁 → 複製其產生的 PowerShell 指令。
2.  **在 VM 執行認證**：RDP 進 VM，以**系統管理員**身分執行 PowerShell，貼上指令。
    - **核心重點**：指令中的 `cmdkey` 會將金鑰存入系統，確保 `sshd` 服務具備存取權。
3.  **驗證連線**：
    ```powershell
    # 確保能看到網路路徑
    ls \\cmuacmustorage.file.core.windows.net\appfiles
    ```

---

## Step 4：安裝 OpenSSH 與建立 SFTP 帳號

### 4-1 安裝 OpenSSH Server
```powershell
# 安裝功能
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# 啟動服務並設為自動啟動
Start-Service sshd
Set-Service -Name sshd -StartupType 'Automatic'
```

### 4-2 建立專屬 SFTP 帳號
```powershell
# 建立帳號（請替換密碼）
$Password = ConvertTo-SecureString "Sftp@Password123!" -AsPlainText -Force
New-LocalUser -Name "sftpuser" -Password $Password -Description "SFTP only account"
```

### 4-3 建立符號連結 (做法二：穩定性關鍵)
將網路 UNC 路徑映射為本地目錄，以支援 `Chroot`。
```powershell
# 建立本地根目錄
mkdir C:\SFTP_Root

# 建立指向 Azure File Share 的符號連結 (UNC 映射)
cmd /c mklink /D C:\SFTP_Root\app \\cmuacmustorage.file.core.windows.net\appfiles
```

### 4-4 設定 NTFS 權限
```powershell
$folder = "C:\SFTP_Root\app"
$acl = Get-Acl $folder
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule("sftpuser", "FullControl", "ContainerInherit,ObjectInherit", "None", "Allow")
$acl.SetAccessRule($rule)
Set-Acl $folder $acl
```

---

## Step 5：配置 OpenSSH 與防火牆

### 5-1 修改 sshd_config
編輯 `C:\ProgramData\ssh\sshd_config`，在檔案**最底部**加入：
```ssh
Match User sftpuser
    # 限制使用者只能在 app 目錄內，且路徑需使用 Windows 絕對路徑格式
    ChrootDirectory C:\SFTP_Root\app
    ForceCommand internal-sftp
    AllowTcpForwarding no
    X11Forwarding no
```
*儲存後重啟服務：`Restart-Service sshd`*

### 5-2 Azure NSG 開放 Port 22
1. 進入 Azure Portal → 找到對應的 **網路安全性群組 (NSG)**。
2. **新增輸入安全性規則**：
   - **目的地連接埠**：`22`
   - **來源**：建議設為您的「辦公室固定 IP」。
   - **名稱**：`Allow-SFTP-SSH`
   - **優先順序**：`290` (優於 RDP)。

---

## Step 6：WinSCP 連線測試

1.  開啟 **WinSCP**。
2.  **新站台設定**：
    - **通訊協定**：SFTP。
    - **主機名稱**：VM 的公用 IP。
    - **使用者名稱**：`sftpuser`。
    - **密碼**：Step 4-2 設定的密碼。
3.  **連線**：第一次連線會提示信任主機金鑰，點擊「是」。
4.  **驗證**：進入後應直接位於 `app` 目錄，且無法往上跳轉。

---

## 常見問題速查 (Troubleshooting)

| 問題現象 | 檢查點 | 解決方式 |
| :--- | :--- | :--- |
| WinSCP 連線逾時 | Azure NSG 防火牆 | 確認 Port 22 已對您的 IP 開放 |
| 登入後報錯或斷開 | Chroot 目錄權限 | 確保 `C:\SFTP_Root` 的所有者為 SYSTEM，且一般使用者無寫入權 |
| 看到目錄但無法寫入 | NTFS 權限 | 重新執行 Step 4-4 的 `Set-Acl` 腳本 |
| 檔案無法同步至 Azure | 認證失效 | 重新執行 Step 3 的 `cmdkey` 指令 |
