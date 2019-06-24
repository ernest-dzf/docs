# flush privileges #

我们经常使用到`flush privileges`，但是什么场合下必须使用`flush privileges`呢？

**当mysql启动时，所有的权限都会被加载到内存中**

如果使用GRANT/REVOKE/SET PASSWORD/RENAME USER命令来更改数据库中的权限表，mysqld服务器将会注意到这些变化，并立即加载更新后的权限表至内存中，即权限生效；

如果使用INSERT/UPDATE/DELETE语句更新权限表，则内存中的权限表不会感知到数据库中权限的更新，必须重启服务器或者使用FLUSH PRIVILEGES命令使更新的权限表加载到内存中，即权限需在重启服务器或者FLUSH PRIVILEGES之后方可生效。



**权限生效的含义**


1. 表级别/列级别的权限，当更新后的权限加载至内存表中，已存在的会话下一次请求时可使用该权限，在修改权限后的建立的会话则立即生效；

2. 数据库级别的权限，当更新后的权限加载至内存表中，已存在会话下一次使用USE db_name后，可使用该权限，在修改权限后的建立的会话则立即生效；

3. 局权限或者修改密码，当更新后的权限加载至内存表中，需要在下一次登录mysqld后，可使用该权限或密码，对已存在会话不起作用。


**举个例子**

	mysql> create user 'dzf'@'%' identified by '199233';
	Query OK, 0 rows affected (0.00 sec)
	
	mysql> quit
	Bye

	[root@VM_0_15_centos data]# mysql -udzf -h172.19.0.15 -p
	Enter password: 
	Welcome to the MySQL monitor.  Commands end with ; or \g.
	Your MySQL connection id is 4391
	Server version: 5.6.43-log MySQL Community Server (GPL)
	
	Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
	
	Oracle is a registered trademark of Oracle Corporation and/or its
	affiliates. Other names may be trademarks of their respective
	owners.
	
	Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

	mysql> quit
	Bye



	mysql> 
	mysql> use mysql;
	Reading table information for completion of table and column names
	You can turn off this feature to get a quicker startup with -A
	
	Database changed
	mysql> update user set password=password('123') where user='dzf';
	Query OK, 1 row affected (0.00 sec)
	Rows matched: 1  Changed: 1  Warnings: 0
	
	mysql> quit
	Bye
	[root@VM_0_15_centos data]# mysql -udzf -h172.19.0.15 -p'123'
	Warning: Using a password on the command line interface can be insecure.
	ERROR 1045 (28000): Access denied for user 'dzf'@'172.19.0.15' (using password: YES)
	[root@VM_0_15_centos data]# 
	


	
	[root@VM_0_15_centos data]# mysql -uroot -p
	Enter password: 
	Welcome to the MySQL monitor.  Commands end with ; or \g.
	Your MySQL connection id is 4396
	Server version: 5.6.43-log MySQL Community Server (GPL)
	
	Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
	
	Oracle is a registered trademark of Oracle Corporation and/or its
	affiliates. Other names may be trademarks of their respective
	owners.
	
	Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
	
	mysql> 
	mysql> use mysql;
	Reading table information for completion of table and column names
	You can turn off this feature to get a quicker startup with -A
	
	Database changed
	mysql> update user set password=password('123') where user='dzf';
	Query OK, 0 rows affected (0.00 sec)
	Rows matched: 1  Changed: 0  Warnings: 0
	
	mysql> flush privileges;
	Query OK, 0 rows affected (0.00 sec)
	
	mysql> quit;
	Bye
	[root@VM_0_15_centos data]# mysql -udzf -h172.19.0.15 -p'123'
	Warning: Using a password on the command line interface can be insecure.
	Welcome to the MySQL monitor.  Commands end with ; or \g.
	Your MySQL connection id is 4397
	Server version: 5.6.43-log MySQL Community Server (GPL)
	
	Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
	
	Oracle is a registered trademark of Oracle Corporation and/or its
	affiliates. Other names may be trademarks of their respective
	owners.
	
	Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
	
	mysql> 
	


可以看到，`create user`之后，没有`flush privileges`，新建的用户也是有效的。

但是`update password set password=password('123') where user='dzf'`之后，如果不`flush privileges`，那么新的密码是无效的。如果想要新的密码生效， 需要重启或者`flush privileges`才可以。