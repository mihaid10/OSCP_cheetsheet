# PsExec

(Microsoft, 2016), https://docs.microsoft.com/en-us/sysinternals/downloads/psexec

PSExec[についてわかりやすいサイト](https://talvinen.hatenablog.com/entry/2017/01/01/171332)

* 横展開（フォーマット）

  ```cmd
  C:\Windows\system32>PsExec.exe [\\<対象ホスト名>] [-u <ユーザ名>] [-p <パスワード>] [<オプション>] <対象ホストに実行させたいコマンド>
  ```

  角括弧で括っている項目に関しては、用途に応じて省略可能

* 横展開（TGTとTGSが登録されている状態）

  PsExecはパスワードハッシュは受け入れない。必ずkerberos認証である必要がある

  ```cmd
  # チケットの確認
  klist
  # 実行
  .\PsExec.exe \\dc01 cmd.exe
  ```

* NTLM認証の場合はIPアドレスを指定

  ```
  psexec.exe \\172.16.179.5 cmd.exe
  ```