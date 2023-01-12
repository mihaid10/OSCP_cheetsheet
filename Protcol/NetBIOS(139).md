# NetBIOS(139)

### nbtscan

有効なNetBIOS名をNetBIOSネームサービスに問い合わせる

```bash
┌──(kali㉿kali)-[~/Documents/OSCP/7.ActiveInformationGathering]
└─$ sudo nbtscan -r 10.11.1.0/24
Doing NBT name scan for addresses from 10.11.1.0/24

IP address       NetBIOS Name     Server    User             MAC address      
------------------------------------------------------------------------------
10.11.1.5        ALICE            <server>  ALICE            00:50:56:ba:1a:c5
10.11.1.101      BREAK            <server>  BREAK            00:00:00:00:00:00
10.11.1.115      TOPHAT           <server>  TOPHAT           00:00:00:00:00:00
10.11.1.136      SUFFERANCE       <server>  SUFFERANCE       00:00:00:00:00:00
10.11.1.231      MAILMAN          <server>  MAILMAN          00:00:00:00:00:00
10.11.1.227      JD               <server>  ADMINISTRATOR    00:50:56:ba:9a:98
----
```

* rオプション：発信UDPポートを137として指定するために使用