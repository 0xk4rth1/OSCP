Target : 10.129.25.190

Started off with the port scan : 22, 80

Adding the ip address in /etc/hosts

10.129.25.190 sea.htb

Enumerating port 80:

*Subdomain Fuzzing using FFUF*

`ffuf -w ~/Seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.orion.htb" -u http://orion.htb/ -fl 87`

![[Screenshot 2026-07-02 at 9.06.32 PM.png]]

And got nothing.

Directory bruteforcing for finding interesting endpoints:

`gobuster dir -u "http://sea.htb" -w ~/SecLists/Discovery/Web-Content/common.txt`

![[Screenshot 2026-07-02 at 9.02.32 PM.png]]

Only positive is application is running php.

`gobuster dir -u "http://sea.htb" -w ~/SecLists/Discovery/Web-Content/common.txt -x php`

but got nothing interesting, tried big.txt (no luck)

then started fuzzing  /themes directory to find more information on the target

we found /bike and that eventually revealed README.md , found it is wondercms

/themes/bike/version -> 3.2.0 (CVE-2023-41425 - XSS to RCE)

Exploiting CVE-2023-41425

Using this exploit : https://github.com/prodigiousMind/CVE-2023-41425/blob/main/exploit.py

