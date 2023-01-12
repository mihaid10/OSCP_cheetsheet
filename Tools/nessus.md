# Nessus

### インストール

Tenableのウェブサイトhttps://www.tenable.com/downloads/nessus

![image-20221229125757214](img/nessus/image-20221229125757214.png)

```bash
┌──(kali㉿kali)-[/usr/share/nmap/scripts]
└─$ sha256sum ~/Downloads/Nessus-10.4.1-ubuntu1404_amd64.deb     
```

```bash
┌──(kali㉿kali)-[/usr/share/nmap/scripts]
└─$ sudo apt install ~/Downloads/Nessus-10.4.1-ubuntu1404_amd64.deb
```

```
sudo systemctl start nessusd.service 
```

https://localhost:8834 



### Basic Network Scan

![image-20221229130046973](img/nessus/image-20221229130046973.png)

![image-20221229130102121](img/nessus/image-20221229130102121.png)

名前に「*** - Basic」とつける必要がありそう

![image-20221229130134185](img/nessus/image-20221229130134185.png)

![image-20221229130150916](img/nessus/image-20221229130150916.png)



### Authenticated Scan

有効なターゲット認証情報を必要とする認証済みスキャンを実行することで、より詳細な情報を生成し、誤検出を減らすことができる

![image-20221229133135296](img/nessus/image-20221229133135296.png)

![image-20221229133143789](img/nessus/image-20221229133143789.png)

![image-20221229133150565](img/nessus/image-20221229133150565.png)



### Scanning with Individual Nessus Plugins

![image-20221229133217489](img/nessus/image-20221229133217489.png)

ホストがOpenしていることがわかっている場合、ホストの検出をオフにする

![image-20221229133223341](img/nessus/image-20221229133223341.png)

openしているポートがわかっていれば絞り込みをする

![image-20221229133231306](img/nessus/image-20221229133231306.png)

![image-20221229133238566](img/nessus/image-20221229133238566.png)