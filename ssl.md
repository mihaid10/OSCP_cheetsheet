### 自己署名証明書の作成

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

鍵を生成したら、証明書とその秘密鍵をファイルにcatする

```
cat bind_shell.key bind_shell.crt > bind_shell.pem
```

