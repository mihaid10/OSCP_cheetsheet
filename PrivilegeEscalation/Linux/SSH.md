# SSH

* kaliで鍵を作る

  ```bash
  ssh-keygen -t rsa -b 4096
  ```

* 攻撃対象ホストで公開鍵をauthorized_keysに登録する

  ```bash
  mkdir .ssh
  chmod 700 .ssh
  cd .ssh
  vi authorized_keys
  chmod 600 authorized_keys
  ```

* 各ディレクトリのパーミッション設定

  ```bash
  ＜接続元（クライアント）の設定例＞
  chmod 755 /home/
  chmod 755 /home/user
  chmod 700 /home/user/.ssh
  chmod 600 /home/user/.ssh/id_rsa
  chmod 644 /home/user/.ssh/id_rsa.pub
  
  ＜接続先（サーバ）の設定例＞
  chmod 755 /home/
  chmod 755 /home/user
  chmod 700 /home/user/.ssh
  chmod 600 /home/user/.ssh/authorized_keys
  ```

* SSHで接続する(from kali)

  ```
  ssh -i .ssh/id_rsa -vvv testuser@172.16.118.128
  ```

  ※ユーザ名は接続先のユーザを指定

* 接続できないときはconfigも確認

  https://soji256.hatenablog.jp/entry/2021/03/04/070000

  ```bash
  # サーバー側
  vi /etc/ssh/sshd_config
  
  PubkeyAuthentication yes
  ```

* 意図的にブロックされていてどうしてもSSHできない時もある。

  ディレクトリのパーミッションも、コンフィグも正しければラビットホールの可能性を疑った方がいいかも
