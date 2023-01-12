# SMTP(25)

* VFRYリクエスト：サーバに電子メールアドレスの検証を要求
* EXPNリクエスト：サーバにメーリングリストのメンバーかどうかを要求

### 接続

```bash
nc -nv 10.11.1.217 25
VRFY root
252 2.0.0 root
VRFY idontexist
550 5.1.1 <idontexist>: Recipient address rejected: User unknown in local recipient table
```

### telnet

以下入力情報

```bash
telnet 192.168.144.55 25
EHLO 192.168.144.55
MAIL FROM: rmurray@victim
rcpt to:tharper@victim
data
To: tharper@victim
From: rmurray@victim
Subject: urgent patch

test mail
http://192.168.119.144/evil.exe
.
quit
```

応答付き

```bash
┌──(kali㉿kali)-[~/Documents/OSCP/13.ClientSideAttacks]
└─$ telnet 192.168.144.55 25                                                               
Trying 192.168.144.55...
Connected to 192.168.144.55.
Escape character is '^]'.
220 VICTIM Microsoft ESMTP MAIL Service, Version: 10.0.17763.1697 ready at  Wed, 4 Jan 2023 00:05:27 -0500 
EHLO 192.168.144.55
250-VICTIM Hello [192.168.119.144]
250-TURN
250-SIZE 2097152
250-ETRN
250-PIPELINING
250-DSN
250-ENHANCEDSTATUSCODES
250-8bitmime
250-BINARYMIME
250-CHUNKING
250-VRFY
250 OK
MAIL FROM: rmurray@victim
250 2.1.0 rmurray@victim....Sender OK
rcpt to:tharper@victim
250 2.1.5 tharper@victim 
data
354 Start mail input; end with <CRLF>.<CRLF>
To: tharper@victim
From: rmurray@victim
Subject: urgent patch

http://192.168.119.144/evil.exe
.
250 2.6.0 <VICTIMTtPsRxnXLD16Q00000001@VICTIM> Queued mail for delivery
quit
221 2.0.0 VICTIM Service closing transmission channel
Connection closed by foreign host.
```

https://qiita.com/awwa/items/dae1f8475c96b506edd0



### メモ

* 一度インターネット接続がないサーバーからメールを送りたくてメールサーバーを構築しようとしたときがあったがDNSサーバーやLDAPサーバーなどが必要であり無理なのでやめること
  https://digitalmania.info/linux/debian-11/build-mail-server/

* 【図解】初心者にもわかるメールの仕組み](https://milestone-of-se.nesuke.com/nw-basic/grasp-nw/mail/)

  ![image-20220904103739143](img/SMTP(25)/image-20220904103739143.png)

  
