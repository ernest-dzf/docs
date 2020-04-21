# general log

开启general log后，将把所有到达mysql server的SQL语句记录下来

## 相关变量



- `show variables like 'general_log'`
  查看是否开启了general log记录。
`general_log`变量可以立即设置，立即生效。
  **这个变量是global的。**
  
  设置了的话，全局生效。
  
  ```mysql
  mysql> show variables like 'general_log';
  +---------------+-------+
  | Variable_name | Value |
  +---------------+-------+
  | general_log   | OFF   |
  +---------------+-------+
  1 row in set (0.00 sec)

  mysql> 

  ```
  
- `show variables like 'general_log_file'`
  查看general log 存储的日志文件是哪个。

  ```mysql
  mysql> show variables like 'general_log_file';
  +------------------+-------------------------------+
  | Variable_name    | Value                         |
  +------------------+-------------------------------+
  | general_log_file | /data/3306/VM_0_15_centos.log |
  +------------------+-------------------------------+
  1 row in set (0.00 sec)
  
  mysql> 
  
  ```

  

- `show variables like 'log_output'`
  `log_output`决定日志的输出位置。可以设置为文件或者输出到表。
  这个变量也是global的。
  可取值为`TABLE`，`FILE`，`NONE`。
  这个变量控制了慢日志和general log的输出位置。
  如果取值为`NONE`，那么就不输出日志。
  `log_output`的取值可以是`TABLE`，`FILE`的组合，比如下面，

  ```mysql
  mysql> show variables like 'log_output';
  +---------------+------------+
  | Variable_name | Value      |
  +---------------+------------+
  | log_output    | FILE,TABLE |
  +---------------+------------+
  1 row in set (0.00 sec)
  
  mysql> 
  
  ```

  这样，日志在表和文件中都有记录。
  如果是记录导表中的话，那么是在`mysql.general_log`和`mysql.slow_log`这两张表中。

## 配置文件中配置开启general log

可以在配置文件中配置开启general log，达到配置持久化的目的。

类似下面这样，

```
[mysqld]
general_log=ON
general_log_file=/data/3306/VM_0_15_centos.log
log_output=TABLE,FILE

```

重启mysql生效。

重启之后，登录查看，

```
mysql> show variables like '%general%';                                                                                                                                     
+------------------+-------------------------------+
| Variable_name    | Value                         |
+------------------+-------------------------------+
| general_log      | ON                            |
| general_log_file | /data/3306/VM_0_15_centos.log |
+------------------+-------------------------------+
2 rows in set (0.00 sec)

mysql> show variables like 'log_output';                                                                                                                                    
+---------------+------------+
| Variable_name | Value      |
+---------------+------------+
| log_output    | FILE,TABLE |
+---------------+------------+
1 row in set (0.00 sec)

mysql> 

```

### general log 内容

```
/usr/local/mysql/bin/mysqld, Version: 5.6.43-log (MySQL Community Server (GPL)). started with:
Tcp port: 3306  Unix socket: /tmp/mysql.sock
Time                 Id Command    Argument
200116 21:25:58    59 Query     show databases
200116 21:26:00    59 Query     SELECT DATABASE()
                   59 Init DB   victor
                   59 Query     show databases
                   59 Query     show tables
                   59 Field List        class 
                   59 Field List        student 
200116 21:26:02    59 Query     show tables
200116 21:26:06    59 Query     select * from class
200116 21:26:08    59 Quit      
200116 21:30:07    60 Connect   root@180.117.109.246 on 
                   60 Connect   Access denied for user 'root'@'180.117.109.246' (using password: YES)
200116 21:31:12    61 Connect   wind@172.19.0.15 on 
                   61 Query     select @@version_comment limit 1
200116 21:31:18    61 Query     show variables like 'general_log_file'
200116 21:31:23    61 Query     show variables like 'general_log'
200116 21:31:30    61 Query     show variables like 'general_log'
200116 21:31:52    61 Query     set @@global.general_log='OFF'

```

