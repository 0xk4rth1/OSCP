
If SNMP is exposed to unauthorized users, it can allow attackers to:

-  Map the internal network
-  Enumerate user accounts
-  Dump configurations
-  Access OS, hardware, and performance details
-  Modify settings (if RW access)

Querying the Full Database/ Dump the Full SNMP Tree:

`snmpwalk -v 2c -c public 10.129.228.102`

![](../Pasted%20image%2020260717115910.png)

(or)

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

Inspect Running Process & Command Line Arguments:

`snmpwalk -v 2c -c public 10.129.228.102 1.3.6.1.2.1.25.4.2.1.5`    (Look closely at the **Processes** section of your `snmpwalk` output)

Linux SNMP agents often expose full command-line arguments, which routinely contain hardcoded passwords, hidden configuration files, or custom scripts running in the background.

**Useful SNMP OIDs for Hacking:**

`1.3.6.1.2.1.1.1.0` OS and version Hostname

`1.3.6.1.2.1.1.5.0` System name Location

`1.3.6.1.2.1.1.6.0` Physical location Uptime

`1.3.6.1.2.1.1.3.0` System uptime User Accounts (Windows)

`1.3.6.1.4.1.77.1.2.25` List of usernames Routing Table

`1.3.6.1.2.1.4.21` Network routes IP Net-to-Media (ARP)

`1.3.6.1.2.1.4.22.1.2` MAC/IP mappings Interface Details

`1.3.6.1.2.1.2.2.1` Ports & traffic stats Storage Stats

`1.3.6.1.2.1.25.2.3.1` Disk usage

**SNMPBrute:** 

we can take now is to use brute force to try different community strings and see if any of them are hidden, instead of using the default value of "public". We can use a tool like SNMPBrute for this purpose.

Tool : 


