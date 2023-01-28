# Metasploit framework

* Metasploit Frameworkの実行結果を保存するためにpostgresql を起動する

  ```bash
  sudo systemctl start postgresql
  ```

* 起動時にサービスを有効にする

  ```bash
  sudo systemctl enable postgresql
  ```

* MSF データベースを作成し、初期化する

  ```bash
  sudo msfdb init
  ```

* Metasploitをアップデートする

  ```bash
  sudo apt update; sudo apt install metasploit-framework
  ```

* msfconsoleを立ち上げる

  ```bash
  sudo msfconsole -q
  ```

* データベースにアクセスして実行結果を確認する

  ```
  services
  services -h
  ```

* nmap.結果をdb登録

  ```
  db_nmap 192.168.149.10 -sT -p 80,135
  ```

* データベース内のコンテンツを整理するためのworkspace

  ```
  workspace
  workspace test
  ```

  ワークスペースの追加と削除は、それぞれ -a と -d を使い、その後にワークスペース名を指定

* モジュールの検索(AuxiliaryとExploitをよく使う)

  ```
  search -h
  search type:auxiliary name:smb
  ```

* 認証情報を収集した結果をデータベースに確認する場合

  ```
  creds
  ```

* モジュール実行時の注意点

  ```
  info
  ```

  でモジュール詳細を確認

  ```
  set target
  ```

  でアーキテクチャを設定。tabで候補が表示される

* payloadはかならずしもmeterpreterを使わなくてよい。普通のリバースシェルでもmsfconsoleで取得できる

  

