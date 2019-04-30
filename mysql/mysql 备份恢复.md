# mysql备份与恢复的几种方式 #

## mysqldump ##

这种方式不仅适用于InnoDB，还适用于其它类型的存储引擎，如MyISAM。

备份的时候将数据库备份成SQL(包含drop，create，insert等语句)，恢复的时候直接导入即可。属于逻辑备份。

比如下面这样，对整个库进行备份。

	[root@VM_0_15_centos ~]# mysqldump -h127.0.0.1 -P3306 -uroot -p'199233' --database victor > backup.sql
	Warning: Using a password on the command line interface can be insecure.
	Warning: Using unique option prefix database instead of databases is deprecated and will be removed in a future release. Please use the full name instead.
	[root@VM_0_15_centos ~]# 

得到的结果是：

	-- MySQL dump 10.13  Distrib 5.6.43, for linux-glibc2.12 (x86_64)
	--
	-- Host: 127.0.0.1    Database: victor
	-- ------------------------------------------------------
	-- Server version       5.6.43
	
	/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
	/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
	/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
	/*!40101 SET NAMES utf8 */;
	/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
	/*!40103 SET TIME_ZONE='+00:00' */;
	/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
	/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
	/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
	/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
	
	--
	-- Current Database: `victor`
	--
	
	CREATE DATABASE /*!32312 IF NOT EXISTS*/ `victor` /*!40100 DEFAULT CHARACTER SET latin1 */;
	
	USE `victor`;
	
	--
	-- Table structure for table `student`
	--
	
	DROP TABLE IF EXISTS `student`;
	/*!40101 SET @saved_cs_client     = @@character_set_client */;
	/*!40101 SET character_set_client = utf8 */;
	CREATE TABLE `student` (
	  `id` int(11) DEFAULT NULL,
	  `name` varchar(32) DEFAULT NULL
	) ENGINE=InnoDB DEFAULT CHARSET=latin1;
	/*!40101 SET character_set_client = @saved_cs_client */;
	
	--
	-- Dumping data for table `student`
	--
	
	LOCK TABLES `student` WRITE;
	/*!40000 ALTER TABLE `student` DISABLE KEYS */;
	INSERT INTO `student` VALUES (1,'victor');
	/*!40000 ALTER TABLE `student` ENABLE KEYS */;
	UNLOCK TABLES;
	/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;
	
	/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
	/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
	/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
	/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
	/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
	/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
	/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;
	
	-- Dump completed on 2019-04-30 19:02:14

上面操作的是对`victor`库进行备份。

对`victor`的单个`student`表进行备份。

	[root@VM_0_15_centos ~]# mysqldump -h127.0.0.1 -P3306 -uroot -p'199233' --database victor --tables class > class.sql


**几个要点：**

1. LOCK TABLES `student` WRITE;

	读写方式锁住表`student`，`lock table`的会话可以对表`student`做修改及查询等操作。而其他会话不能对该表做任何操作，包括select也要被阻塞。

	session-1：

		mysql> lock tables student write;
		Query OK, 0 rows affected (0.00 sec)
		
		mysql> desc student;
		+-------+-------------+------+-----+---------+-------+
		| Field | Type        | Null | Key | Default | Extra |
		+-------+-------------+------+-----+---------+-------+
		| id    | int(11)     | YES  |     | NULL    |       |
		| name  | varchar(32) | YES  |     | NULL    |       |
		+-------+-------------+------+-----+---------+-------+
		2 rows in set (0.00 sec)
		
		mysql> insert into `student` values(2, 'kate');
		Query OK, 1 row affected (0.01 sec)
		
		mysql> 
		
	session-2：

		mysql> select * from student;
		+------+--------+
		| id   | name   |
		+------+--------+
		|    1 | victor |
		+------+--------+
		1 row in set (0.00 sec)
		
		mysql> select * from student;
	
	session-2会被卡住。	

2. UNLOCK TABLES;

	LOCK TABLES为当前会话锁定表。 UNLOCK TABLES释放被当前会话持有的任何锁。

3. LOCK TABLES `student` READ;
	
	只读方式锁住`student`，该表只能被select，不能被修改。如果在lock时，该表上存在事务，则lock语句挂起，直到事务结束。

	执行 `LOCK TABLES student READ` 的当前session也不能执行对该表的写入操作。

		mysql> lock tables student read;
		Query OK, 0 rows affected (0.00 sec)
		
		mysql> insert into `student` values(4, 'kevin');
		ERROR 1099 (HY000): Table 'student' was locked with a READ lock and can't be updated
		mysql> 
		
