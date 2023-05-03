# Isakmp(UDP500)

[toc]

https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting

### 認証方式の確認

```bash
ike-scan -M 10.129.228.122
```

![image-20230501093927140](img/isakmp(500UDP)/image-20230501093927140.png)

AuthがPSK(preshared key)になっている。攻撃者にとって良い兆候

最後の行が大事(1 returned handshakeがあると攻撃できる可能性がある)

* *0 returned handshake; 0 returned notify:* This means the target is **not an IPsec gateway**.

* **1 returned handshake; 0 returned notify:\*** This means the **target is configured for IPsec and is willing to perform IKE negotiation, and either one or more of the transforms you proposed are acceptable** (a valid transform will be shown in the output).

* *0 returned handshake; 1 returned notify:* VPN gateways respond with a notify message when **none of the transforms are acceptable** (though some gateways do not, in which case further analysis and a revised proposal should be tried).

### 認証情報の取得

#### Brute-force

https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting

#### SNMP

snmpの全体検索でハッシュ値が見つかる場合がある

[snmpの全体検索](./SNMP(161).md/#全体検索（コミュニティ文字列は大体public)

![image-20230501094426785](img/isakmp(500UDP)/image-20230501094426785.png)

ハッシュ値が見つかったら[Crackstation](./../PasswordAttacks.md/#CrackStation)/johnでクラックする

### VPNの設定

#### strongswanのインストール

https://www.tecmint.com/setup-ipsec-vpn-with-strongswan-on-debian-ubuntu/

![image-20230501095102139](img/isakmp(500UDP)/image-20230501095102139.png)

#### ipsec.secretsの設定

```
sudo vi /etc/ipsec.secrets

---
10.129.228.122 %any : PSK "Dudecake1!"
```

![image-20230501095201548](img/isakmp(500UDP)/image-20230501095201548.png)

#### ipsec.confの設定

```bash
sudo vi /etc/ipsec.conf

------
conn conceal
	type=transport
	keyexchange=ikev1
	left=10.10.14.75
	leftprotoport=tcp
	right=10.129.228.122
	rightprotoport=tcp
	authby=psk
	esp=3des-sha1
	fragmentation=yes
	ike=3des-sha1-modp1024
	ikelifetime=8h
	auto=start
```

* Type: host-to-hostの通信のため`transport`

* vpnはikev1(鍵交換)を使用しています。私たちのIPアドレスが左の接続（左）、ConcealのIPアドレスが右の接続（右）
* 認証はPSK(authby)
* LifeDurationは、「python3 -c 'print(int("0x00007080", 16))'」というコマンドで変換する必要があります。しかし、このコマンドは秒単位で値を返すので、時間に変換すると8時間になってしまいます。

#### IPsecの起動

```bash
sudo ipsec start --nofork
sudo ipsec restart --nofork
```

![image-20230501095301558](img/isakmp(500UDP)/image-20230501095301558.png)

一番下の文字列が出たらOK

