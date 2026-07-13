
**Git Dumper** : https://github.com/arthaud/git-dumper

`python3 git-dumper.py http://example.com/.git/ <foldername>`

![](../Pasted%20image%2020260707095023.png)
 
 (Even though the application shows /.git/ folder is not accessible - 403)
 
SQLite3 :

To read .db file: `sqlite3 database.db`
To retrieve tables : `.tables`
To dump the entire table : `select * from users;`

SQLMAP:

For **GET** Parameter : `sqlmap -u "https://example.com?u=*"`
For **POST** Parameter: `sqlmap -r req --batch`

Find and list all databases: `sqlmap -r req --dbs`
List tables within  a specific database: `sqlmap -r req -D <database_name> --tables`
To List columns inside a specific table: `sqlmap -r req -D <database_name> -T <table_name> --columns`
To Dump the table data : `sqlmap -r req -D <database_name> -T <table_name> --dump `

MYSQL : 

To connect mysql database: `mysql -u <username> -p `

```
show databases;
use <database_name>;

show tables;
select * from <table_name>;
```

Postgresql :

To connect postgresql database: `psql -h localhost -p 5432 -U postgres`

```
\l                      (to list all available databases)
\c <database_name>      (to select a database)
\dt                     (to list all tables)

SELECT * from users; 
```

