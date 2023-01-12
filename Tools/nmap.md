# nmap

### TCPコネクトスキャン

```
nmap -sT 10.0.0.1
```

3wayハンドシェイク実行され、ＴＣＰコネクションが張られるため、  
firewall,IDS,IPS等に検知されやすい。



### ステルススキャン

```
nmap -sS 10.0.0.1
```

SYN AYN/ACK RST のみ秘匿性が高い。



### Networksweeping

```bash
kali@kali:~$ nmap -v -sn 10.11.1.1-254 -oG ping-sweep.txt
kali@kali:~$ grep Up ping-sweep.txt | cut -d " " -f 2
```



### ポートを指定してSweep

```bash
kali@kali:~$ nmap -p 80 10.11.1.1-254 -oG web-sweep.txt
kali@kali:~$ grep open web-sweep.txt | cut -d" " -f2
```



### TCP TOP20ポートスキャン

```bash
kali@kali:~$ nmap -sT -A --top-ports=20 10.11.1.1-254 -oG top-port-sweep.txt

# TOP20の根拠
kali@kali:~$ cat /usr/share/nmap/nmap-services 
```



### OS Fingerprinting

返送されたパケットを検査することで、ターゲットのOSを推測しようとする。

OSによってTCP/IPスタックの実装が微妙に異なることが多く（デフォルトのTTL値やTCPウィンドウサイズが異なるなど）、こうしたわずかな差異でフィンガープリントを作成する

```bash
kali@kali:~$ sudo nmap -O 10.11.1.220
```



### サービスバナー(特定のポートで動作しているサービスを特定)

```bash
nmap -sV -sT -A 10.11.1.220
```

* サービスバナー（-sV）を検査
* OSやサービスの列挙スクリプト（-A）



### NSE

* ヘルプ表示

```bash
/usr/share/nmap/scripts
kali@kali:~$ nmap --script-help dns-zone-transfer
kali@kali:~$ nmap --script-help=clamav-exec.nse
```

```
nmap 10.11.1.220 --script=smb-os-discovery
```



* 脆弱性スクリプトを全適用

```bash
kali@kali:~$ sudo nmap --script vuln 10.11.1.10
```

