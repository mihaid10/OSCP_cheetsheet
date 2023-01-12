# DIRB

```bash
dirb http://www.megacorpone.com -r -z 10
```

* -r：非再帰的なスキャン 
* -z 10：各リクエストに10ミリ秒の遅延を追加する



# Gobuster

```bash
gobuster dir -u http://127.0.0.1:4444 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -t 40
```

