# nmap

[toc]

### サービスバナー(特定のポートで動作しているサービスを特定)

```bash
nmap -sV -sT -A 10.11.1.220 -T4
```

```bash
nmap -p 21,80,81,135,139,445,808,1433,5985,15567,32843,32844,32846,47001,49664-49670 -sCV -oA nmap/nmap-tcpscripts 10.10.10.59
```

* サービスバナー（-sV）を検査
* OSやサービスの列挙スクリプト（-A）

```
nmap -sT -p- 10.11.1.220 -T4
```

### UDPスキャン

```bash
sudo nmap -sU -v --top-ports=20 10.11.1.118
sudo nmap -sU -v -p 1-1000 10.11.1.111
```

※ UDPは時間がかかるのでportを適宜絞ること

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



### ポートスキャン

```
sudo nmap -sC -sS -p0-65535 sandbox.local
```

* -sC：デフォルトのスクリプトセット
* -sS：時間を短縮するためのSYNスキャン



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



* SMTP関連のスクリプトを全実行

  ```bash
  nmap 10.11.1.72 -p 25 --script=smtp-*
  ```

* 脆弱性スクリプトを全適用

```bash
kali@kali:~$ sudo nmap --script vuln 10.11.1.10 -T4
```



適正なイーサネット接続の環境にいる場合は`-T4`オプションを付与することでnmapの高速化が期待できる

