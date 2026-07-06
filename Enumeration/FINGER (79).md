
Connecting the service:

`ncat -nv <ip> 79`

Enumerating users:  (https://github.com/pentestmonkey/finger-user-enum)

`finger-user-enum.pl ~/SecLists/Usernames/xato-net-10-million-usernames.txt -t $ip -p 79`

