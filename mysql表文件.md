# mysql表文件 #

mysql 中创建数据库，会在数据目录下面创建相关目录，比如下面，

	# root @ localhost in /usr/local/mysql/data [6:04:34] 
	$ ls
	auto.cnf    client-cert.pem  ib_buffer_pool  ib_logfile1                localhost.localdomain.pid  private_key.pem  server-key.pem
	ca-key.pem  client-key.pem   ibdata1         ibtmp1                     mysql                      public_key.pem   sys
	ca.pem      fenqu            ib_logfile0     localhost.localdomain.err  performance_schema         server-cert.pem  victor
	
	# root @ localhost in /usr/local/mysql/data [6:04:35] 
	$ cd victor
	
	# root @ localhost in /usr/local/mysql/data/victor [6:04:53] 
	$ pwd
	/usr/local/mysql/data/victor
	
	# root @ localhost in /usr/local/mysql/data/victor [6:04:54] 
	$ ls
	db.opt  employees.frm  employees.ibd
	
	# root @ localhost in /usr/local/mysql/data/victor [6:04:55] 
	$       

上面对应的数据库如下：

	mysql> show databases;
	+--------------------+
	| Database           |
	+--------------------+
	| information_schema |
	| fenqu              |
	| mysql              |
	| performance_schema |
	| sys                |
	| victor             |
	+--------------------+
	6 rows in set (0.01 sec)
	
	mysql> use victor;
	Reading table information for completion of table and column names
	You can turn off this feature to get a quicker startup with -A
	
	Database changed
	mysql> show tables;
	+------------------+
	| Tables_in_victor |
	+------------------+
	| employees        |
	+------------------+
	1 row in set (0.00 sec)
	
	mysql> 

可以看到数据库`victor`在数据目录`/usr/local/mysql/data`下对应的目录就是`victor`。目录项目有三个文件，分别是`db.opt`、`employees.frm`和`employees.ibd`。

## employees.frm文件 ##
## employees.ibd文件 ##
## db.opt文件 ##

先来看看db.opt文件长啥样

	# root @ localhost in /usr/local/mysql/data/victor [6:24:18] 
	$ ls
	db.opt  employees.frm  employees.ibd
	
	# root @ localhost in /usr/local/mysql/data/victor [6:24:24] 
	$ cat db.opt 
	default-character-set=latin1
	default-collation=latin1_swedish_ci
	
	# root @ localhost in /usr/local/mysql/data/victor [6:24:27] 
	$ 

可以看到有两行，分别是`default-character-set=latin1`和`default-collation=latin1_swedish_ci`。

作用是用来记录该库的默认字符集编码和字符集排序规则。

如果你创建数据库指定默认字符集和排序规则，那么后续创建的表如果没有指定字符集和排序规则，那么该新建的表将采用db.opt文件中指定的属性。比如下面：

	mysql> show create database cha;
	+----------+--------------------------------------------------------------+
	| Database | Create Database                                              |
	+----------+--------------------------------------------------------------+
	| cha      | CREATE DATABASE `cha` /*!40100 DEFAULT CHARACTER SET utf8 */ |
	+----------+--------------------------------------------------------------+
	1 row in set (0.00 sec)
	mysql> create table person(id int not null auto_increment,
	    ->     name varchar(8),
	    ->     birthday datetime,
	    ->     constraint pk__person primary key(id));
	Query OK, 0 rows affected (0.02 sec)		
	mysql> show tables;
	+---------------+
	| Tables_in_cha |
	+---------------+
	| person        |
	+---------------+
	1 row in set (0.00 sec)
	
	mysql> show create table person\G
	*************************** 1. row ***************************
	       Table: person
	Create Table: CREATE TABLE `person` (
	  `id` int(11) NOT NULL AUTO_INCREMENT,
	  `name` varchar(8) DEFAULT NULL,
	  `birthday` datetime DEFAULT NULL,
	  PRIMARY KEY (`id`)
	) ENGINE=InnoDB DEFAULT CHARSET=utf8
	1 row in set (0.00 sec)
	
	mysql> 
，

	# root @ localhost in /usr/local/mysql/data/cha [6:31:27] 
	$ ls
	db.opt  person.frm  person.ibd
	
	# root @ localhost in /usr/local/mysql/data/cha [6:35:26] 
	$ cat db.opt
	default-character-set=utf8
	default-collation=utf8_general_ci
	
	# root @ localhost in /usr/local/mysql/data/cha [6:35:30] 
	$ 

可以看到 `cha` 库默认字符集是utf8，那么创建表`person`没有指定字符集的话，就使用db.opt文件指定的字符集。