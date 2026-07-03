
To find the default windows AV status (below we're checking windows defender)

`sc query windefend`                ( sc stands for service control )

![](../Pasted%20image%2020260702113514.png)

(check the STATE, it is RUNNING, so AV is running)

To find all the services running in the machine. (we can use this to find other antiviruses, other than windows defender)

`sc queryex type= service `

To get more details on the firewall:

`netsh advfirewall firewall dump`
`netsh firewall show state`
`netsh firewall show config`