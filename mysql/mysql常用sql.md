# mysql常用sql #
## 用户管理类 ##
1. CREATE USER '用户'@'主机' IDENTIFIED BY '密码'
	
	创建一个用户：

		mysql> create user 'wind'@'%' identified by '199233';
		Query OK, 0 rows affected (0.09 sec)
		
		mysql> select user,host from mysql.user where user='wind'\G
		*************************** 1. row ***************************
		user: wind
		host: %
		1 row in set (0.11 sec)
		
		mysql> 

2. SHOW GRANTS FOR '用户'@'主机'
	
	展示某一个用户所拥有的权限：

		mysql> show grants for 'victor'@'%';
		+------------------------------------+
		| Grants for victor@%                |
		+------------------------------------+
		| GRANT USAGE ON *.* TO 'victor'@'%' |
		+------------------------------------+
		1 row in set (0.00 sec)

3. `GRANT ALL PRIVILEGES ON *.* TO '用户'@'主机' WITH GRANT OPTION`

	对账户授权。这里需要注意一下上面的`*.*`,表示对所有db的所有表进行授权，`*`号不能被反单引号括起来。当然也可以对某个db的某个表进行授权。单个db或者表这时候是可以被反单引号括起来的。

		mysql> grant all privileges on `fenqu`.* to kate@'%';
		Query OK, 0 rows affected (0.10 sec)
		
		mysql> show grants for kate;
		+-------------------------------------------------+
		| Grants for kate@%                               |
		+-------------------------------------------------+
		| GRANT USAGE ON *.* TO 'kate'@'%'                |
		| GRANT ALL PRIVILEGES ON `fenqu`.* TO 'kate'@'%' |
		+-------------------------------------------------+
		2 rows in set (0.00 sec)

4. SET PASSWORD=PASSWORD('123')

	重置账户密码。

		mysql> set password=password('1234qwer()');
		Query OK, 0 rows affected, 1 warning (0.00 sec)
		
		mysql> quit
		Bye
		
		# root @ localhost in ~ [0:48:33] 
		$ mysql -uvictor -p'1234qwer()'
		mysql: [Warning] Using a password on the command line interface can be insecure.
		Welcome to the MySQL monitor.  Commands end with ; or \g.
		Your MySQL connection id is 16
		Server version: 5.7.23 MySQL Community Server (GPL)
		
		Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.
		
		Oracle is a registered trademark of Oracle Corporation and/or its
		affiliates. Other names may be trademarks of their respective
		owners.
		
		Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
		
		mysql> 

## 系统管理类 ##

1. 查看占用实际空间大小

    	SELECT  table_schema AS "Database", ROUND(SUM(data_length + index_length + data_free) / 1024 / 1024, 2)  AS "Size (MB)"  FROM  information_schema.TABLES  GROUP BY  table_schema;