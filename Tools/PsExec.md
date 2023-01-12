# PsExec

(Microsoft, 2016), https://docs.microsoft.com/en-us/sysinternals/downloads/psexec

* 横展開（TGTとTGSが登録されている状態）

  PsExecはパスワードハッシュは受け入れない。必ずkerberos認証である必要がある

  ```cmd
  # チケットの確認
  klist
  # 実行
  .\PsExec.exe \\dc01 cmd.exe
  ```

* 