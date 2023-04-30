# PortScan

```bash
#!/bin/bash
host=10.5.5.11
for port in {1..65535}; do
    timeout .1 bash -c "echo >/dev/tcp/$host/$port" &&
        echo "port $port is open"
done
echo "Done"
```

```
for i in {1..10}; do echo 10.11.1.$i;done
```

```
nc -zv 192.168.11.xxx 1-65535
```

```cmd
C:\wamp64\attendance\images\nc.exe -nzvv -w 1 10.10.44.142 1-6000
nc.exe -nzvv 192.168.11.xxx 1-65535

-e prog        接続後に起動するアプリ [危険との事]
-g gateway     ルーティングゲートウェー指定（最大８つ）
-G num         ルーティングポインタ指定（4,8,12...）
-h             ヘルプ表示
-i secs        データ出力やポートスキャンの間隔を秒で指定
-l             リッスンモード
-n             DNSの名前解決を行わない
-o file                 ファイルを16進で出力する
-p port                 （リッスンモードの）ポート番号指定
-r                      接続先ポートをランダムに指定する
-s addr                 リッスンするIPアドレスを指定。ローカルで設定されたもの
-t                      telnetプロトコルで通信する
-u                      UDPモード指定
-v                      情報表示モード。2回指定でより詳細になる
-w secs                 タイムアウト指定
-z                      ゼロI/Oモード（ポートスキャン時）

```

※ lateralmoveの時は必要なポートのみ検索にしないと一生終わらない

