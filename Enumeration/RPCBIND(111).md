
To dump all registered RPC programs 

`rpcinfo -p <TARGET-IP>`

To display clean output

`rpcinfo -s <TARGET-IP>`

Enumerating rpcbind using nmap:

`nmap -p 111 --script=rpcinfo,rpc-grind <TARGET-IP>`

