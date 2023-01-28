## windows

* ログインユーザ

```
whoami
net user student
```

* システム上のほかのユーザアカウント

```
net user
```

* ホスト名の調査

  ```
  hostname
  ```

* システム情報の列挙

  ```
  systeminfo
  systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
  ```

  ※OSバージョンでgoogle検索して直接脆弱性を探す方法もあり

* プロセスとサービスの列挙

  ```
  tasklist /SVC
  ```

  ※特権ユーザによって実行されているプロセスをリストアップしない

* ネットワーク情報

  ```
  ipconfig /all
  ```

* ネットワークのルーティングテーブル表示

  ```
  route print
  ```

* アクティブなネットワーク接続を表示

  ```
  netstat -ano
  ```

  -a：アクティブなTCPコネクション

  -n：アドレスとポートを数字で表示

  -o：各コネクションのオーナーPIDを表示

* ファイアーウォールプロファイルの検査

  ```
  netsh advfirewall show currentprofile
  netsh advfirewall firewall show rule name=all
  ```

* スケジュールタスクの表示

  ```
  schtasks /query /fo LIST /v
  ```

  /query：タスクを表示

  /FO LIST：出力形式を単純なリストに設定

  /V：冗長な出力を要求

* Windowsインストーラーによってインストールされるアプリの一覧表示

  ```
  wmic product get name, version, vendor
  ```

* システム全体の更新情報を一覧表示

  ```
  wmic qfe get Caption, Description, HotFixID, InstalledOn
  ```

  HotFixIDとInstalledOnの情報を組み合わせることで、対象となるWindowsオペレーティングシステムのセキュリティ状況を正確に示すことができる

* 書き込み可能なファイルの列挙

  ```
  accesschk.exe -uws "Everyone" "C:\Program Files"
  ```

  * -u：エラーを抑制
  * -w：書き込みアクセス権を検索
  * -s：再帰的な検索を実行

* 書き込み可能なファイルの列挙（PowerShell)

  ```powershell
  Get-ChildItem "C:\Program Files" -Recurse | Get-ACL | ?{$_.AccessToString -match "Everyone\sAllow\s\sModify"}
  ```

  * Get-Acl ：指定されたファイルまたはディレクトリのすべての権限を取得する

  * Get-ChildItem：ディレクトリ配下のすべてを列挙

* マウントドライブ一覧（接続されていないものも含む）

  ```
  mountvol
  ```

* デバイスドライバとカーネルモジュールの取得

  ```powershell
  powershell
  driverquery.exe /v /fo csv | ConvertFrom-CSV | Select-Object ‘Display Name’, ‘Start Mode’, Path
  ```

  /fo csv：CSV形式での出力を要求

* ドライバのバージョン番号を取得

  ```powershell
  Get-WmiObject Win32_PnPSignedDriver | Select-Object DeviceName, DriverVersion, Manufacturer | Where-Object {$_.DeviceName -like "*VMware*"}
  ```

* レジストリ設定「AlwaysInstallElevated48」の確認

  ※このキーが有効 (1 に設定) の場合、すべてのユーザーが Windows Installer パッケージを昇格特権で実行できるようになる

  ```cmd
  c:\Users\student>reg query HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer
  ```

  ```cmd
  c:\Users\student>reg query HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\Installer
  ```

* 所属グループの確認(整合性レベルの確認が可能)

  ```
  whoami /groups
  ```

* ユーザのパスワード変更

  ```
  net user admin Ev!lpass
  ```
  
* ローカルグループの表示

  ```
  net localgroup
  ```

* ローカルグループのメンバーの表示

  ```
  net localgroup Administrators
  ```

* グローバルなグループの表示

  ```
  net group
  ```

  ![image-20230113074128746](img/InformationGathering/image-20230113074128746.png)

  https://atmarkit.itmedia.co.jp/ait/articles/0609/02/news014_2.html#_ga=2.160874941.1084612904.1673563106-525287863.1651237399
  
* ログインユーザのSID確認

  ```
  whoami /user
  ```

* 管理共有の確認

  ```
  net share
  ```

  

## linux

* ログインユーザ

```
whoami
id
```

* ユーザ列挙

```
cat /etc/passwd
```

* ホスト名の調査

```
hostname
```

* OS情報の確認

  ```bash
  cat /etc/issue
  cat /etc/*-release
  uname -a
  ```

* プロセスの列挙

  ```
  ps axu
  ```

* ネットワーク表示

  ```
  ifconfig
  ip
  ```

* ネットワークのルーティングテーブル

  ```
  /sbin/route
  ```

* アクティブなネットワークの情報

  ```
  ss -anp
  ```

  -a：すべての接続を表示

  -n：ホスト名解決を回避し

  -p：接続が属しているプロセス名を表示

* ファイウォールルールの表示

  ```
  iptables
  ```

  ※root権限が必要

* Cronの確認

  ```
   ls -lah /etc/cron* 
   cat /etc/crontab
   cat /var/log/cron.log
  ```

* インストールされたアプリケーション一覧の表示

  ```
  dpkg -l
  ```

* 書き込み可能なファイルの列挙

  ```
  find / -writable -type d 2>/dev/null
  ```

* マウントファイルシステムの列挙

  ```cmd
  mount
  # 起動時にマウントされるすべてのドライブ一覧
  cat /etc/fstab
  ```

* 利用可能なすべてのディスクの表示

  ```bash
  /bin/lsblk
  ```

* ロードされたカーネル・モジュールを列挙

  ```
  lsmod
  ```

* 特定のモジュールの詳細表示

  ```
  /sbin/modinfo libata
  ```

* SUIDファイルの検索

  ```
  find / -perm -u=s -type f 2>/dev/null
  ```

  

