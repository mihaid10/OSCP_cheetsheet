# Leveraging Unquoted Service Paths

多くの場合、サードパーティ製ソフトウェアに付随するサービスは、`C:\Program Files` ディレクトリに格納される。

名前にスペースが入っていた場合悪用の可能性がある

あるサービスを`C:\Program Files\My Program\My Service\service.exe.` のようなパスに保存しているとする。

サービスパスが引用符で囲まれていない場合、Windowsはサービスを起動するたびに、次のパスから実行可能ファイルを実行しようとする。

```
C:\Program.exe
C:\Program Files\My.exe
C:\Program Files\My Program\My.exe
C:\Program Files\My Program\My service\service.exe
```

`Program Files`配下は書き込み権限がない場合が多く、ソフトウェアのメインディレクトリ（`C:\Program Files\My Program`）やサブディレクトリ（`C:\Program Files\My Program\My service`）が誤って設定され、悪意のある My.exe バイナリを仕込むことができる可能性が⾼くなる。

