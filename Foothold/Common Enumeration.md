
**For getting information about the system:**

`systeminfo`

**For getting the host machine name:**

`hostname`

**To know about the patching information list using wmic:**

(windows management instrumentation commandline - quick fix engineering = wmic qfe)

`wmic qfe`

we can get informations like knowledge base id (kbid), HotFixID, InstalledBy, InstalledOn

**To find information about logical disk information:**

`wmic logicaldisk get caption,description,providername`

**To get the username information:**

`whoami`

**To check the privileges:**

`whoami /priv`

![](../Pasted%20image%2020260702102757.png)

To get the group information for the user (only shows the groups that the currently logged-in user belongs to)

`whoami /groups`

**To get all informations about the user, groups, privileges belong to the user:**

`whoami /all`

**To find the users on the machine:**

`net user`

![](../Pasted%20image%2020260702103446.png)

(Even though if we have service account access, still we can find the users in the machine, and can move laterally)

**To find all the domain users** 

`net user /dom`

**To get individual user information**

`net user <username>`  :  " net user babis "

**To get the localgroup information**

`net localgroup
`net localgroup administrators`          (we can find all the users belong to the group)

**To get information about the network:**

`ipconfig /all`           
`arp -a`             (to list the arp tables)
`route print`    (to map internal networks and find pivoting paths to hidden subnets)

**To check all the ports (listening):**

`netstat -ano`      (identifying internal ports are crucial for port forwarding and accessing it outside)

`ss -tulnp`

Permission Check:

`icacls "c:/path/to/file.txt"` (To check, whether the user has read,write,execute permission of a file )

```
-(F): Full Control (Includes complete edit, modify, and delete rights).
-(M): Modify Access (Allows reading, writing, and deleting the file).
-(W): Write Access (Allows editing and adding data).
-(RX)or (R): Read-Only (You **cannot** edit the file)
```

![](../Screenshot%202026-07-20%20at%209.49.18%20PM.png)

To modify permission, if we are the owner of a file, we can modify its permission using cacls

`cacls root.txt \t \e \p <username>:F`

![](../Pasted%20image%2020260724121917.png)

`Get-ACL <file or folder name> | fl *`   (fl stands for format list)  (powershell)

![](../Pasted%20image%2020260724123437.png)

![](../Pasted%20image%2020260724123903.png)
