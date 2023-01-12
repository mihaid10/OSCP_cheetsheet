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

