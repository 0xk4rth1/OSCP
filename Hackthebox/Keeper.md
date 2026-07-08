Target : 10.129.229.41

Started off with the port scan : 22, 80

Enumerating Port : 80

Need to add keeper.htb in /etc/hosts

sudo nano /etc/hosts

**Subdomain Fuzzing** revealed a another subdomain called tickets.keeper.htb

tickets                 [Status: 200, Size: 4236, Words: 407, Lines: 154, Duration: 280ms]

![](../Screenshot%202026-07-04%20at%2010.23.34%20AM.png)

**Directory Bruteforcing:** 

in keeper.htb, we got nothing. 

but in tickets.keeper.htb we got some interesting endpoints. 

/index.html           (Status: 200) [Size: 4236]
/m                    (Status: 200) [Size: 2309]
/rte                  (Status: 200) [Size: 95]
/rtf                  (Status: 200) [Size: 95]
/rt  

And the applicaiton is running request tracker, i looked out for default credentials which is 

`root : password`

There is another user in the request tracker application : lnorgaard@keeper.htb

While exploring the application, found the ticket number #30000 which seems interesting, but for accessing it, we need to get in to `lnorgaard` this user account

got the credentials for `lnorgaard : Welcome2023!` form the users portal.
![](../Screenshot%202026-07-04%20at%2010.57.09%20AM.png)

I used this credentials to get initial foothold in lnorgaard user

```
lnorgaard@keeper:~$ cat user.txt
a5968d82866437a7cf773f05163b2cfb
```

While searching about keepass .dmp file we got to know about the `CVE-2023-32784`

Using this exploit : https://github.com/vdohney/keepass-password-dumper

we got the master password : `rødgrød med fløde`

Then, connected with the keypass client and got the ssh password:  `F4><3K0nd!`

![](../Screenshot%202026-07-04%20at%2012.21.04%20PM.png)

But also, got the putty-gen private key in the notes. 

```
nano id.ppk
puttygen id.ppk -O private-openssh -o id_rsa
chmod 600 id_rsa

ssh root@keeper.htb -i id_rsa
```

![](../Screenshot%202026-07-04%20at%2012.19.57%20PM.png)

And finally got shell as user `root`

`root.txt : edc7ba50113b26c5392a04d8bb01c3a6`


**To be noted:**

1. Got the user flag without any hints.
2. Even, had a clear escalation path view, but struggled finding working exploit, even after got the password, it not seemed like one. so i ignored it. have to cross check it.
