# FTP

[toc]

```
ftp <ipアドレス>
# ログイン情報がない時はとりあえずanonymousで試す
ls
passive
binary
put xxxx.exe
bye
```

[パッシブモードとアクティブモード](https://48n.jp/blog/2016/06/17/learn-ftp/)

binary：バイナリモードで転送. exeファイルを配送するときは必ずbinaryモードを指定すること

```
ftp> binary
200 Type set to I.
ftp> put accesschk.exe 
```

FTPサーバーのファイルを再帰的に取得する

```bash
wget -r 'ftp://ftp_user:UTDRSCH53c"$6hys@10.129.1.183'
```

![image-20230501192434145](img/FTP(21)/image-20230501192434145.png)

