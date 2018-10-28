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

	
3. GRANT ALL PRIVILEGES ON *.* TO '用户'@'主机' WITH GRANT OPTION

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
