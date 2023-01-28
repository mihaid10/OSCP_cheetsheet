# PrivilegeCheck

* PrivescCheck.ps1

```
powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck"
```

https://github.com/itm4n/PrivescCheck

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

  