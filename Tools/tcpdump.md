# tcpdump

### pcapの解析

```
sudo tcpdump -r password_cracking_filtered.pcap
```



### 簡易フィルター（ipアドレスでフィルタ)

```
sudo tcpdump -n -r password_cracking_filtered.pcap | awk -F" " '{print $5}' | sort | uniq -c | head
```



### src host/ dst hostフィルタ

```
sudo tcpdump -n src host 172.16.40.10 -r password_cracking_filtered.pcap
sudo tcpdump -n dst host 172.16.40.10 -r password_cracking_filtered.pcap
sudo tcpdump -n port 81 -r password_cracking_filtered.pcap
```



### トラフィックダンプ

```
sudo tcpdump -nX -r password_cracking_filtered.pcap

...
08:51:25.043062 IP 208.68.234.99.33313 > 172.16.40.10.81: Flags [P.], seq 1:140, ack 1
  0x0000:  4500 00bf 158c 4000 3906 9cea d044 ea63  E.....@.9....D.c
  0x0010:  ac10 280a 8221 0051 a726 a77c 6fd8 ee8a  ..(..!.Q.&.|o...
  0x0020:  8018 0073 1c76 0000 0101 080a 0185 b2f2  ...s.v..........
  0x0030:  0441 f5e3 4745 5420 2f2f 6164 6d69 6e20  .A..GET.//admin.
  0x0040:  4854 5450 2f31 2e31 0d0a 486f 7374 3a20  HTTP/1.1..Host:.
  0x0050:  6164 6d69 6e2e 6d65 6761 636f 7270 6f6e  admin.megacorpon
  0x0060:  652e 636f 6d3a 3831 0d0a 5573 6572 2d41  e.com:81..User-A
  0x0070:  6765 6e74 3a20 5465 6820 466f 7265 7374  gent:.Teh.Forest
  0x0080:  204c 6f62 7374 6572 0d0a 4175 7468 6f72  .Lobster..Author
  0x0090:  697a 6174 696f 6e3a 2042 6173 6963 2059  ization:.Basic.Y
  0x00a0:  5752 7461 5734 3662 6d46 7562 3352 6c59  WRtaW46bmFub3RlY
  0x00b0:  3268 7562 3278 765a 336b 780d 0a0d 0a    2hub2xvZ3kx....
...
```

* -X オプション：パケットデータを HEX と ASCIIの両方のフォーマットで出力



### 高度なフィルタリング

ACKとPSHは4バイト目の4ビット目と5ビット目で表現されている

![image-20221226114115684](img/tcpdump/image-20221226114115684.png)

```
echo "$((2#00011000))"
sudo tcpdump -A -n 'tcp[13] = 24' -r password_cracking_filtered.pcap
```



