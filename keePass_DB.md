# KeePass DB

[toc]

bitwardenのようなOSSのパスワード管理ツール。主にwindowsで用いられる

kdbxがDBとなっている

![image-20230501192617045](img/keePass_DB/image-20230501192617045.png)

### マスターパスワードのクラック

```
keepass2john User/Tim/Files/tim.kdbx
keepass2john User/Tim/Files/tim.kdbx > tim.kdbx.hash
```

![image-20230501192743237](img/keePass_DB/image-20230501192743237.png)

```
john --wordlist=/usr/share/wordlists/rockyou.txt tim.kdbx.hash 
```

![image-20230501192813477](img/keePass_DB/image-20230501192813477.png)

### kpcliでパスワードを取得する

```
kpcli --kdb User/Tim/Files/tim.kdbx
```

<img src="img/keePass_DB/image-20230501192955758.png" alt="image-20230501192955758" style="zoom:50%;" />

```
find .
show -f 2
```

<img src="img/keePass_DB/image-20230501193026323.png" alt="image-20230501193026323" style="zoom:50%;" />

