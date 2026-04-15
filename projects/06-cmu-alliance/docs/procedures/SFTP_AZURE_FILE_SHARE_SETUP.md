# Windows VM SFTP 與 Azure File Share 權限設定指南 (標準規範)

本文件定義「本機 SFTP → Windows VM → Azure File Share」架構的標準設定規範。為了確保 `ChrootDirectory` 穩定性與服務帳號存取權，統一採用**「SYSTEM 帳戶掛載 + 符號連結」**做法。

## 1. 核心權限問題與限制

在 Windows 上使用 OpenSSH (Win32-OpenSSH) 存取 Azure File Share 時，必須符合以下限制：

- **ChrootDirectory 不支援網路路徑**：OpenSSH 在 Windows 上僅支援本機 NTFS 路徑。
- **Session 0 隔離**：OpenSSH 服務 (`sshd`) 以 `SYSTEM` 帳號執行，無法看到一般使用者掛載的網路磁碟（如 `Z:`）。
- **NTFS 權限要求**：SFTP 使用者帳號（如 `sftpuser`）必須具備目標目錄的 NTFS 讀寫權限。

---

## 2. 標準設定步驟 (做法二)

這是目前最穩定且支援嚴格目錄限制 (Chroot) 的實作方式。

### 步驟 A：以 SYSTEM 權限掛載 Azure File Share
由於 `sshd` 服務執行於 `SYSTEM` 帳戶，必須確保該帳戶能存取網路分享。

1.  下載 [Sysinternals PsExec](https://learn.microsoft.com/en-us/sysinternals/downloads/psexec)。
2.  以系統管理員身分開啟 CMD，執行以下指令以 `SYSTEM` 權限啟動一個新的 CMD 視窗：
    ```cmd
    psexec -i -s cmd.exe
    ```
3.  在**彈出的 SYSTEM CMD 視窗**中執行掛載（請替換為您的 Storage Account 資訊）：
    ```cmd
    net use Z: \\storageaccount.file.core.windows.net\sharename /u:AZURE\storageaccount 您的密鑰 /persistent:yes
    ```
    *注意：務必使用 `/persistent:yes` 確保重啟後自動重新掛載。*

### 步驟 B：建立本地符號連結 (Symbolic Link)
將網路磁碟映射至本機 NTFS 路徑，以便 OpenSSH 識別。

1.  在本地硬碟（如 `C:`）建立 SFTP 根目錄（需嚴格限制寫入權限）：
    ```cmd
    mkdir C:\SFTP_Root
    ```
2.  建立指向網路磁碟特定資料夾的符號連結：
    ```cmd
    mklink /D C:\SFTP_Root\app Z:\app
    ```

### 步驟 C：修改 `sshd_config` 設定
編輯 `C:\ProgramData\ssh\sshd_config`，針對使用者套用設定：

```ssh
Match User sftpuser
    ChrootDirectory C:\SFTP_Root\app
    ForceCommand internal-sftp
    AllowTcpForwarding no
    X11Forwarding no
```

---

## 3. 權限修復與安全性

### A. 給予 SFTP 帳號 NTFS 權限
確保 `sftpuser` 具備對 `Z:` 磁碟中資料夾的完整存取權：

```powershell
# PowerShell (以系統管理員身分)
$folder = "Z:\app"
$acl = Get-Acl $folder
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "sftpuser", "FullControl", "ContainerInherit,ObjectInherit", "None", "Allow"
)
$acl.SetAccessRule($rule)
Set-Acl $folder $acl
```

### B. 目錄安全性規範
為了讓 `ChrootDirectory` 運作，**`C:\SFTP_Root` 及其所有父層目錄**的權限必須：
1.  擁有者為 `SYSTEM` 或 `Administrators`。
2.  **不允許**一般使用者（包括 `sftpuser`）具備寫入權限。

---

## 4. 問題排查

| 問題現象 | 可能原因 | 解決方法 |
| :--- | :--- | :--- |
| 連線成功但目錄空白 | `sshd` 看不到 `Z:` | 確認掛載是在 `SYSTEM` 權限下執行的（步驟 A） |
| 連線後立即被中斷 | `ChrootDirectory` 路徑非本機路徑 | 確認使用的是符號連結路徑（步驟 B） |
| 寫入失敗 (Access Denied) | NTFS 權限不足 | 執行 `Set-Acl` 腳本（步驟 3.A） |
| 無法掛載網路磁碟 | Azure 認證不正確 | 檢查 Storage Account Key 是否有效 |

---

## 5. 維護建議
- **重啟服務**：每次修改 `sshd_config` 後，請重啟服務：`Restart-Service sshd`。
- **自動化掛載**：若重啟後磁碟未自動掛載，建議將「步驟 A」的 `net use` 指令寫入排程工作 (Scheduled Task)，並設定以 `SYSTEM` 帳號執行。
