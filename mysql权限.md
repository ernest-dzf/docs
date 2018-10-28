# mysql权限 #
## all privileges权限 ##

经常看到 all privileges权限，这个all privileges 权限具体有哪些权限呢？

平时创建账号的时候，可以分为两大类，一类是业务系统的账号，基于具体的数据库上面做操作。一类是管理员账号，会涉及到 mysql、information_schema、performance_schema。

### 创建测试用数据库 ###
所以创建一个新的数据库，模拟业务数据库。

	mysql> create database devops;
	Query OK, 1 row affected (0.53 sec)
	
	mysql> show databases;
	+--------------------+
	| Database           |
	+--------------------+
	| information_schema |
	| Victor             |
	| cha                |
	| devops             |
	| fenqu              |
	| mysql              |
	| performance_schema |
	| sys                |
	| victor             |
	+--------------------+
	9 rows in set (0.00 sec)
	
	mysql> 

### 创建用于测试的用户，并赋权all privileges ###
分别创建基于“业务”和基于“管理员”的所有权限`all privileges`。

	mysql> grant all privileges on devops.* to ops1@'%' identified by 'devops1';
	Query OK, 0 rows affected (0.00 sec)
	
	mysql> grant all privileges on devops.* to ops2@'%' identified by 'devops2' with grant  option;
	Query OK, 0 rows affected (0.00 sec)
	
	mysql> grant all privileges on *.* to ops3@'%' identified by 'devops3';
	Query OK, 0 rows affected (0.00 sec)
	
	mysql> grant all privileges on *.* to ops4@'%' identified by 'devops4' with grant option ;
	Query OK, 0 rows affected (0.00 sec)

注意上面这种用法，grant的时候，如果带上了`identified by '密码'`，那么即使该用户不存在，也会先创建该用户，然后grant赋权。

检查一下刚刚创建的用户的权限，如下：


	mysql> show grants for ops1;
	+--------------------------------------------------+
	| Grants for ops1@%                                |
	+--------------------------------------------------+
	| GRANT USAGE ON *.* TO 'ops1'@'%'                 |
	| GRANT ALL PRIVILEGES ON `devops`.* TO 'ops1'@'%' |
	+--------------------------------------------------+
	2 rows in set (0.00 sec)
	
	mysql> show grants for ops2;
	+--------------------------------------------------------------------+
	| Grants for ops2@%                                                  |
	+--------------------------------------------------------------------+
	| GRANT USAGE ON *.* TO 'ops2'@'%'                                   |
	| GRANT ALL PRIVILEGES ON `devops`.* TO 'ops2'@'%' WITH GRANT OPTION |
	+--------------------------------------------------------------------+
	2 rows in set (0.00 sec)
	
	mysql> show grants for ops3;
	+-------------------------------------------+
	| Grants for ops3@%                         |
	+-------------------------------------------+
	| GRANT ALL PRIVILEGES ON *.* TO 'ops3'@'%' |
	+-------------------------------------------+
	1 row in set (0.00 sec)
	
	mysql> show grants for ops4;
	+-------------------------------------------------------------+
	| Grants for ops4@%                                           |
	+-------------------------------------------------------------+
	| GRANT ALL PRIVILEGES ON *.* TO 'ops4'@'%' WITH GRANT OPTION |
	+-------------------------------------------------------------+
	1 row in set (0.00 sec)
	
	mysql> 


从上面看到大家显示的都是all privilges，实际看不出来什么，所以我们可以反向考虑。我回收一个基本的select 权限。看看剩余的权限都有哪些。

为啥这样呢。可以把all privileges 看成一个整体，拿走一个就不是整体了，那就会把其余的全部列出来展现。

### revoke权限 ###

先revoke权限。
	
	mysql> revoke select on devops.* from 'ops1'@'%' ;
	Query OK, 0 rows affected (0.00 sec)
	
	mysql> revoke select on devops.* from 'ops2'@'%' ;
	Query OK, 0 rows affected (0.00 sec)
	
	mysql> revoke select on *.* from 'ops3'@'%' ;
	Query OK, 0 rows affected (0.10 sec)
	
	mysql> revoke select on *.* from 'ops4'@'%' ;
	Query OK, 0 rows affected (0.00 sec)
	
	mysql> 

### 再次检查账户权限 ###

然后再次检查所有账户的权限，如下：

	mysql> show grants for ops1\G
	*************************** 1. row ***************************
	Grants for ops1@%: GRANT USAGE ON *.* TO 'ops1'@'%'
	*************************** 2. row ***************************
	Grants for ops1@%: GRANT INSERT, UPDATE, DELETE, CREATE, DROP, REFERENCES, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, EVENT, TRIGGER ON `devops`.* TO 'ops1'@'%'
	2 rows in set (0.00 sec)
	
	mysql> show grants for ops2\G
	*************************** 1. row ***************************
	Grants for ops2@%: GRANT USAGE ON *.* TO 'ops2'@'%'
	*************************** 2. row ***************************
	Grants for ops2@%: GRANT INSERT, UPDATE, DELETE, CREATE, DROP, REFERENCES, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, EVENT, TRIGGER ON `devops`.* TO 'ops2'@'%' WITH GRANT OPTION
	2 rows in set (0.00 sec)
	
	mysql> show grants for ops3\G
	*************************** 1. row ***************************
	Grants for ops3@%: GRANT INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, SHUTDOWN, PROCESS, FILE, REFERENCES, INDEX, ALTER, SHOW DATABASES, SUPER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER, CREATE TABLESPACE ON *.* TO 'ops3'@'%'
	1 row in set (0.00 sec)
	
	mysql> show grants for ops4\G
	*************************** 1. row ***************************
	Grants for ops4@%: GRANT INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, SHUTDOWN, PROCESS, FILE, REFERENCES, INDEX, ALTER, SHOW DATABASES, SUPER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER, CREATE TABLESPACE ON *.* TO 'ops4'@'%' WITH GRANT OPTION
	1 row in set (0.00 sec)
	
	mysql> 

### 总结 ###

可以看到，基于`*.*`的all privileges 比基于某个库的all privileges多了一些权限。

	

	RELOAD, SHUTDOWN, PROCESS, FILE, SHOW DATABASES, SUPER, REPLICATION SLAVE, REPLICATION CLIENT, CREATE USER, CREATE TABLESPACE 

## mysql所有的权限 ##

先来一张图，便于记忆，总共28个权限。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/mysql%E6%9D%83%E9%99%90.png)

### 4张权限表 ###

mysql中存在4个控制权限的表，分别为**user**表，**db**表，**tables_priv**表，**columns_priv**表。

mysql权限表的验证过程为：

1. 先从user表中的Host,User,Password这3个字段中判断连接的ip、用户名、密码是否存在，存在则通过验证
2. 通过身份认证后，进行权限分配，按照user，db，tables_priv，columns_priv的顺序进行验证。即先检查全局权限表 user，如果user中对应的权限为Y，则此用户对所有数据库的权限都为Y，将不再检查db, tables_priv,columns_priv；如果为N，则到db表中检查此用户对应的具体数据库，并得到db中为Y的权限；如果db中为N，则检 查tables_priv中此数据库对应的具体表，取得表中的权限Y，以此类推

### 具体权限说明 ###