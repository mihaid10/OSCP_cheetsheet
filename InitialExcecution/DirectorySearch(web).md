httpとhttpsは別々にスキャンすること

# DIRB

```bash
dirb http://www.megacorpone.com -r -z 10
```

* -r：非再帰的なスキャン 
* -z 10：各リクエストに10ミリ秒の遅延を追加する



# Gobuster

```
gobuster dir --help
```

```bash
gobuster dir -u http://127.0.0.1:4444 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -t 40
```

* CGIがあるwebserver用のwordlistで実行

```bash
gobuster dir -u http://10.11.1.71/ -w /usr/share/wordlists/dirb/vulns/cgis.txt -s '200,204,403,500' -b ''
```

-s：Positive status codes
-e：Expanded mode, print full URLs

```
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u https://10.10.10.7/ -k
```

-k：SSL認証をスキップ

* リダイレクトが大量に発生するときサイズで除外する方法

```bash
gobuster dir -u http://10.11.1.133/1f2e73705207bdd6467e109c1606ed29-21213/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 40 --exclude-length 323
```

