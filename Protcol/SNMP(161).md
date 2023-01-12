# SNMP(161)

### Simple Network Management Protocol

* UDPベース
* ネットワークを管理するプロトコル
* 一般的に使用されているSNMPプロトコル1、2、2cは、トラフィックの暗号化を行わないため、ローカルネットワーク上でSNMPの情報と認証情報を簡単に傍受することができる
* 従来のSNMPプロトコルは、認証スキームも弱く、デフォルトのパブリックおよびプライベートコミュニティー文字列で設定されるのが一般的



###  The SNMP MIB Tree

SNMP Management Information Base (MIB) は、通常ネットワーク管理に関連する情報を含むデータベース

データベースは、ツリーのように構成され、枝はさまざまな組織やネットワーク機能を表す

#### MIB値一覧

|                        |                  |
| :--------------------: | ---------------- |
|  1.3.6.1.2.1.25.1.6.0  | System Processes |
| 1.3.6.1.2.1.25.4.2.1.2 | Running Programs |
| 1.3.6.1.2.1.25.4.2.1.4 | Processes Path   |
| 1.3.6.1.2.1.25.2.3.1.4 | Storage Units    |
| 1.3.6.1.2.1.25.6.3.1.2 | Software Name    |
| 1.3.6.1.4.1.77.1.2.25  | User Accounts    |
|  1.3.6.1.2.1.6.13.1.3  | TCP Local Ports  |



### ポートスキャン(nmap)

```bash
sudo nmap -sU --open -p 161 10.11.1.1-254 -oG open-snmp.txt
grep open open-snmp.txt | cut -d" " -f2
```



### ポートスキャン(onesixtyone)

こちらで見つかったIPアドレスはその後の情報収集コマンドが刺さりやすい。刺さる量は少ない。

```bash
kali@kali:~$ echo public > community
kali@kali:~$ echo private >> community
kali@kali:~$ echo manager >> community

kali@kali:~$ for ip in $(seq 1 254); do echo 10.11.1.$ip; done > ips

kali@kali:~$ onesixtyone -c community -i ips

----
┌──(kali㉿kali)-[~/Documents/OSCP/7.ActiveInformationGathering]
└─$ onesixtyone -c community -i ips 
```



### snmpwalk(MIB値の検索ツール)

#### 全体検索（コミュニティ文字列は大体public)

```bash
snmpwalk -c public -v1 -t 10 10.11.1.227
```

* -cオプション：コミュニティ文字列を指定する
* -vオプション：SNMPバージョン番号を指定する
* -t 10：タイムアウト時間を10秒に増やす



#### windowsユーザ列挙

```bash
snmpwalk -c public -v1 10.11.1.227 1.3.6.1.4.1.77.1.2.25
```



#### Windows Process列挙

```bash
snmpwalk -c public -v1 10.11.1.227 1.3.6.1.2.1.25.4.2.1.2
```



#### TCP open ポート列挙

```bash
snmpwalk -c public -v1 10.11.1.227 1.3.6.1.2.1.6.13.1.3
```



#### インストール済みソフトウェア列挙

```bash
snmpwalk -c public -v1 10.11.1.227 1.3.6.1.2.1.25.6.3.1.2
```



### snmp-check
