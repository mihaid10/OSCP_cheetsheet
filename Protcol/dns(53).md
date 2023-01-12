# dns

### 用語集

![image-20221228104216170](img/dns(53)/image-20221228104216170.png)

* ネームサーバー：**ゾーン情報を管理する**サーバー

  * プライマリDNSサーバー：ゾーン情報の大本を管理

  * セカンダリDNSサーバー：ゾーン情報のコピーを管理

    ![image-20221228104048276](img/dns(53)/image-20221228104048276.png)

* 権威DNSサーバー：ネームサーバーと同義

* コンテンツサーバー：ネームサーバーと同義

* リゾルバサーバー

  * スタブリゾルバ：クライアントからDNSサーバーへの問い合わせを行う

  * フルサービスリゾルバサーバー：ほかのDNSサーバーへの問い合わせを再帰的に行い、名前解決を行った結果を返す

    8.8.8.8はグーグルが提供するフルサービスリゾルバサーバ

* キャッシュサーバー：リゾルバサーバーと同義



### ドメイン情報収集(dnsrecon)

```bash
dnsrecon -d megacorpone.com -t axfr

# ドメインリスト攻撃
dnsrecon -d megacorp.com -D ./list.txt -t brt 
```

### ドメイン情報収集（dig)

```bash
dig @192.168.51.14 axfr future-gadget.lab.
```

### ドメイン情報収集 (host)

```bash
host www.megacorpone.com  
host -t mx megacorpone.com
host -t txt megacorpone.com

# ネームサーバーの取得
$ host -t ns megacorpone.com | cut -d " " -f 4     

# ゾーン情報要求
$ host -l megacorpone.com ns2.megacorpone.com
$ host -l <ドメイン名> <ネームサーバー名>

# 逆引き
for ip in $(seq 50 100); do host 51.222.169.$ip; done | grep -v "not found"
# ※逆引きはゾーン情報のPTRにIPアドレスの記載がない限りヒットしない。
```

### ドメイン情報収集 (DNSenum)

ブルートフォースまでまとめて実施してくれる

```bash
dnsenum zonetransfer.me
```



### nmap script

```bash
nmap --script=dns-zone-transfer -p 53 ns2.megacorpone.com
nmap --script=dns-zone-transfer -p 53 <ネームサーバ>
```



### IPアドレス群からDNSサーバを見つけゾーン情報列挙

* nmapで53ポートをOPENしているサーバーを見つける

  ```
  nmap -sC -p53 192.168.175.0/24
  ```

  [ルートサーバー](https://www.google.co.jp/search?q=root-servers.net+%E3%81%A8%E3%81%AF&sxsrf=ALiCzsajjF5dt8N_-UocYYyhBJnscmLzOw%3A1672186899697&source=hp&ei=E4yrY5_DKISk2roPuLWb8AI&iflsig=AJiK0e8AAAAAY6uaI9FRW43cS5rasYpBQDN4QSWCrn56&oq=root-servers.net.&gs_lcp=Cgdnd3Mtd2l6EAEYATIJCAAQgAQQDRATMgkIABAeEPEEEBMyCQgAEB4Q8QQQEzIJCAAQHhDxBBATMgkIABAeEPEEEBMyCAgAEB4QDRATMg0IABAFEB4Q8QQQChATMgsIABAFEB4Q8QQQEzILCAAQBRAeEPEEEBMyCwgAEAUQHhDxBBATUABYAGDgEmgAcAB4AIABYogBYpIBATGYAQCgAQKgAQE&sclient=gws-wiz)

* DNSサーバーを見つけたらドメイン名を取得する

  ```bash
  host <dnsサーバーのip> <dnsサーバーのip>
  ```

* ゾーン情報要求する

  ```
  dig axfr <ドメイン名> @<dnsサーバーのip>
  ```

  * AXFR：すべてのゾーン情報を同期
  * IXFR：差分転送

### resolve.conf編集

* resolve.confを編集可能にする

  ```bash
  sudo chattr -i /etc/resolv.conf
  ```

* resolve.confを編集する

  ```bash
  sudo vi /etc/resolv.conf
  
  # 編集例
  ---
  search localdomain za.tryhackme.loc
  nameserver 10.200.61.101
  nameserver 8.8.8.8
  # Shorten name resolution timeouts to 1 second
  options timeout:1
  # only attempt to resolve a hostname 2 times
  options attempts:2
  ---
  ```

* resolve.confを編集不可にする

  ```
  sudo chattr -i /etc/resolv.conf
  ```

  

## ゾーンファイル

```bash
@ IN SOA ns.example.nflabs. mail.example.nflabs. (
    YYYYMMDDxx  ; SERIAL
    28800       ; REFRESH
    7200        ; RETRY
    864000      ; EXPIRE
    86400       ; MINIMUM
)

www     IN    A 192.168.22.10
```

* @ と入力することでゾーン名を指定できる
* SOAレコード：権威を持つゾーンの開始点を示す
* SOAレコードの中身：
  * ns.example.nflabs.: このゾーンのプライマリサーバの名前。FQDN表記
    (MNAME)
  * mail.example.nflabs.: このゾーンの管理者のメールアドレス。FQDN表記(RNAME)
  * REFRESH：セカンダリサーバがプライマリサーバにゾーン情報を確認する時
    間間隔。32ビットで示す。
  * RETRY：セカンダリサーバがゾーン情報を更新できなかった場合に再更新する時間間隔。32ビットで示す。
  * EXPIRE：ゾーン情報の更新ができない状態が続いた場合、セカンダリサーバ
    のデータを継続利用できる最大時間。32ビットで示す
  * MINIMUM：ネガティブキャッシュのTTL。32ビットで示す。ネガティブ
    キャッシュとは、存在しないドメイン名であるという情報のキャッシュのこ
    と
* `www.example.nflabs`と`192.168.22.10`を関連づけている。Aレコードを用いて記入する。末尾にドットを付けず`www`と記載する場合`www.example.nflabs.`と解釈される。