# SSH

* sshdの設定の確認

  ```bash
  grep -v '^#' /etc/ssh/sshd_config | uniq
  ```

* タイムアウトを設定してssh

  ```bash
  ssh -o ConnectTimeout=10 gibson@10.11.1.71
  ```

* 王道のSSH

  ```bash
  ssh -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no" student@192.168.127.52 -p 2222
  ```

  



### Bad keys

認証鍵が一致している場合、この公開鍵を持つデバイスにアクセスするために使用することができる
https://github.com/rapid7/ssh-badkeys



### nmap script

```
ls -lh /usr/share/nmap/scripts/*ssh*

----
-rw-r--r-- 1 root root 1.2K  1月 18  2022 /usr/share/nmap/scripts/ssh-auth-methods.nse
-rw-r--r-- 1 root root 3.0K  1月 18  2022 /usr/share/nmap/scripts/ssh-brute.nse
-rw-r--r-- 1 root root  16K  1月 18  2022 /usr/share/nmap/scripts/ssh-hostkey.nse
-rw-r--r-- 1 root root 5.9K  1月 18  2022 /usr/share/nmap/scripts/ssh-publickey-acceptance.nse
-rw-r--r-- 1 root root 3.7K  1月 18  2022 /usr/share/nmap/scripts/ssh-run.nse
-rw-r--r-- 1 root root 5.3K  1月 18  2022 /usr/share/nmap/scripts/ssh2-enum-algos.nse
-rw-r--r-- 1 root root 1.4K  1月 18  2022 /usr/share/nmap/scripts/sshv1.nse
```

```bash
nmap 10.11.1.71 -p 22 -sV --script=ssh-hostkey
```

