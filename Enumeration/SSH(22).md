
To connect SSH:

```
ssh -i <private_key> <username>@<host>

ssh <username>@<host>

ssh -o HostKeyAlgorithms=+ssh-rsa <username>@<hostname>

```

**Generating SSH keypairs**

`ssh-keygen -t ed25519
`
ssh-keygen`

`mv ed25519.pub ~/.ssh/authorized_keys`

Forwarding local ports through SSH

``ssh <username>@<host> -L 8080:127.0.0.1:8080``

**Sharing Files through SCP**

Upload a file: 

`scp  /path/to/local/file.txt  <username>@<host>:/path/to/remote/directory/`

Download a file: 

`scp  <username>@<host>:/path/to/remote/file.txt  /path/to/local/directory/`

Share an Entire Folder:  

`scp -r  /path/to/local/folder <username>@<host>:/path/to/remote/directory/`
