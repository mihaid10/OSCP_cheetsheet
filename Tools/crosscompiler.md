# CrossCompile



# mingw

### インストール

```
kali@kali:~$ sudo apt install mingw-w64
```



### コンパイル

WindowsPEファイルにコンパイルする

```bash
i686-w64-mingw32-gcc 42341.c -o syncbreeze_exploit.exe -lws2_32
ls -lah
```



### windowsクライアントのコンパイラを利用する場合

```cmd
C:\Program Files\mingw-w64\i686-7.2.0-posix-dwarf-rt_v5-rev1> mingw-w64.bat

C:\Program Files\mingw-w64\i686-7.2.0-posix-dwarf-rt_v5-rev1>echo off
Microsoft Windows [Version 10.0.10240]
(c) 2015 Microsoft Corporation. All rights reserved.

C:\> gcc
gcc: fatal error: no input files
compilation terminated.

C:\> gcc --help
Usage: gcc [options] file...
Options:
  -pass-exit-codes         Exit with highest error code from a phase.
  --help                   Display this information.
  --target-help            Display target specific command line options.
  --help={common|optimizers|params|target|warnings|[^]{joined|separate|undocumented}}[
                           Display specific types of command line options.
  (Use '-v --help' to display command line options of sub-processes).
  --version                Display compiler version information.
...
```

![image-20230106102344123](img/mingw(crosscompiler)/image-20230106102344123.png)



## Linuxの32bitバイナリを作成する

https://forums.offensive-security.com/showthread.php?33885-Rooted-Comments-on-cross-compiling-binaries-for-32-bit-Linux-OS-and-a-small-hint

* gcc-multilibが必要

  ないと以下のようなエラーになる。

  ```bash
  gcc -m32 -pthread dirtyc0w2.c -o dirty -lcrypt
  
  In file included from /usr/include/features.h:392,
                   from /usr/include/fcntl.h:25,
                   from dirtyc0w2.c:31:
  /usr/include/features-time64.h:20:10: fatal error: bits/wordsize.h: そのようなファイルやディレクトリはありません
     20 | #include <bits/wordsize.h>
        |          ^~~~~~~~~~~~~~~~~
  compilation terminated.  
  ```

  ```bash
  1. apt-get install gcc-9-base libgcc-9-dev libc6-dev
  2. apt-get install gcc-multilib
  ```

* staticを付ける

  staticを付けない場合共有ライブラリの`/usr/lib32`などを参照しにいく。ファイル名は常に.soで終わる。

* lddコマンドを用いることで依存している共有ライブラリを特定できる

  ```bash
  ldd *binary
  ```

* 古いカーネル用のバイナリをコンパイルしている場合、次のようなエラーがでることがある

  ```bash
  ./shared-binary: error while loading shared libraries: requires glibc 2.5 or later dynamic linker
  ```

  このエラーが表示された場合、実行ファイルをコンパイル・リンクする際に、以下のコマンドライン引数を含める必要があることを意味します。

  ```
  -Wl,--hash-style=both
  ```

  
