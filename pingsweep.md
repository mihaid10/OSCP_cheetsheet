# Ping Sweep

### ping

```bash
for i in {1..254} ;do (ping -c 1 192.168.22.$i | grep "bytes from" &) ;done
```

### ping(windows)

```cmd
for /L %i in (1,1,255) do @ping -n 1 -w 200 172.16.250.%i > nul && echo 172.16.250.%i is up.
```



### Netcat

```bash
#!/bin/bash

for ip in $(seq 1 254); do
  nc -n -v -z -w 1 192.168.11.$ip 80 2>&1 |grep open
done
```

```bash
for ip in $(seq 1 254); do nc -n -v -z -w 1 192.168.11.$ip 80 2>&1 |grep open; done
```



### Nmap

```bash
kali@kali:~$ nmap -p 80 10.11.1.1-254 -oG web-sweep.txt
kali@kali:~$ grep open web-sweep.txt | cut -d" " -f2
```



### curl

```
for ip in $(seq 1 254); do curl http://172.18.0.$ip/ 2>&1 ; done
```

