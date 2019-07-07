# mysql主备复制 #


## GTID
从mysql 5.6开始，复制有两种实现方式，基于binlog和基于GTID。

## show master logs

查看主db的binlog文件。

	mysql root@localhost:victor> show master logs;
	+------------+-----------+
	| Log_name   | File_size |
	+------------+-----------+
	| bin.000001 | 65429     |
	| bin.000002 | 1193884   |
	| bin.000003 | 174       |
	| bin.000004 | 1195      |
	| bin.000005 | 214       |
	| bin.000006 | 214       |
	| bin.000007 | 214       |
	| bin.000008 | 214       |
	| bin.000009 | 214       |
	| bin.000010 | 191       |
	| bin.000011 | 2576      |
	+------------+-----------+
	11 rows in set
	Time: 0.015s
	mysql root@localhost:victor>


## binlog的pos

经常说的binlog的偏移到底是啥含义呢？其实就是binlog文件的字节偏移。

比如我们在一个binlog文件中的末尾看到类似这样的东西：

	……
    593 #190703  2:02:53 server id 1234567  end_log_pos 65429 CRC32 0x038a3adf  Stop                         
    594 DELIMITER ;
    595 # End of log file
    596 ROLLBACK /* added by mysqlbinlog */;
    597 /*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;                                                 
    598 /*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;

可以看到`end_log_pos`是65429。其实这也是这个binlog文件的大小。

	[root@victor2 3306]# ls -l bin.000001 
	-rw-rw---- 1 mysql mysql 65429 7月   3 02:02 bin.000001
	[root@victor2 3306]# 
	




## binlog

使用`hexdump`解析任意一个binlog文件，可以返现魔法数字。比如下面。

	hexdump -Cv bin.000001|less

结果是：	

	0000000  fe 62 69 6e 4c 9c 1b 5d  0f 87 d6 12 00 74 00 00  |.binL..].....t..|                               
	00000010  00 78 00 00 00 00 00 04  00 35 2e 36 2e 34 34 2d  |.x.......5.6.44-|                               
	00000020  6c 6f 67 00 00 00 00 00  00 00 00 00 00 00 00 00  |log.............|                               
	00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|                               
	00000040  00 00 00 00 00 00 00 00  00 00 00 4c 9c 1b 5d 13  |...........L..].|                               
	00000050  38 0d 00 08 00 12 00 04  04 04 04 12 00 00 5c 00  |8.............\.|                               
	00000060  04 1a 08 00 00 00 08 08  08 02 00 00 00 0a 0a 0a  |................|                               
	00000070  19 19 00 01 9e fb a7 e4  4c 9c 1b 5d 02 87 d6 12  |........L..]....|                               
	00000080  00 eb 06 00 00 63 07 00  00 00 00 01 00 00 00 00  |.....c..........|                               
	00000090  00 00 00 05 00 00 22 00  00 00 00 00 00 01 00 00  |......".........|                               
	000000a0  00 00 00 00 00 00 06 03  73 74 64 04 2d 00 2d 00  |........std.-.-.|                               
	000000b0  2d 00 0c 01 6d 79 73 71  6c 00 6d 79 73 71 6c 00  |-...mysql.mysql.|                               
	000000c0  43 52 45 41 54 45 20 54  41 42 4c 45 20 49 46 20  |CREATE TABLE IF |                               
	000000d0  4e 4f 54 20 45 58 49 53  54 53 20 64 62 20 28 20  |NOT EXISTS db ( |                               
	000000e0  20 20 48 6f 73 74 20 63  68 61 72 28 36 30 29 20  |  Host char(60) |                               
	000000f0  62 69 6e 61 72 79 20 44  45 46 41 55 4c 54 20 27  |binary DEFAULT '|                               
	00000100  27 20 4e 4f 54 20 4e 55  4c 4c 2c 20 44 62 20 63  |' NOT NULL, Db c|                               
	00000110  68 61 72 28 36 34 29 20  62 69 6e 61 72 79 20 44  |har(64) binary D|                               
	00000120  45 46 41 55 4c 54 20 27  27 20 4e 4f 54 20 4e 55  |EFAULT '' NOT NU|                               
	00000130  4c 4c 2c 20 55 73 65 72  20 63 68 61 72 28 31 36  |LL, User char(16|                               
	00000140  29 20 62 69 6e 61 72 79  20 44 45 46 41 55 4c 54  |) binary DEFAULT|                               
	00000150  20 27 27 20 4e 4f 54 20  4e 55 4c 4c 2c 20 53 65  | '' NOT NULL, Se|                               
	00000160  6c 65 63 74 5f 70 72 69  76 20 65 6e 75 6d 28 27  |lect_priv enum('|  
	
这里前四个字节固定是`fe 62 69 6e`。



binlog由一系列的binlog event构成。每个binlog event包含header和data两部分。 

- header部分提供的是event的公共的类型信息，包括event的创建时间，服务器等等。
- data部分提供的是针对该event的具体信息，如具体数据的修改。
- 从mysql5.0版本开始，binlog采用的是v4版本，第一个event都是format_desc event 用于描述binlog文件的格式版本，这个格式就是event写入binlog文件的格式。
- 接下来的event就是按照上面的格式版本写入的event。

- 最后一个rotate event用于说明下一个binlog文件。
- binlog索引文件是一个文本文件，其中内容为当前的binlog文件列表。



接下来分析下几种常见的event，其他的event类型可以参见官方文档。event数据结构如下：

	+=====================================+
	| event  | timestamp         0 : 4    |
	| header +----------------------------+
	|        | type_code         4 : 1    |
	|        +----------------------------+
	|        | server_id         5 : 4    |
	|        +----------------------------+
	|        | event_length      9 : 4    |
	|        +----------------------------+
	|        | next_position    13 : 4    |
	|        +----------------------------+
	|        | flags            17 : 2    |
	|        +----------------------------+
	|        | extra_headers    19 : x-19 |
	+=====================================+
	| event  | fixed part        x : y    |
	| data   +----------------------------+
	|        | variable part              |
	+=====================================+

## 搭建 ##

两台机器，分别是`10.0.0.2`（victor2）和`10.0.0.7`（victor1）。

`victor1`的mysql的配置如下：

	[mysqld]
	
	########basic settings########
	server-id = 1234568 
	port = 3306
	user = mysql
	
	autocommit = 1
	character_set_server=utf8mb4
	#skip_name_resolve = 1
	max_connections = 10
	max_connect_errors = 3
	datadir = /data/3306
	
	explicit_defaults_for_timestamp = 1
	#join_buffer_size = 134217728
	#tmp_table_size = 67108864
	tmpdir = /tmp
	max_allowed_packet = 16777216
	sql_mode = "STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,NO_ZERO_DATE,NO_ZERO_IN_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER"
	interactive_timeout = 1800
	wait_timeout = 1800
	#read_buffer_size = 16777216
	#read_rnd_buffer_size = 33554432
	#sort_buffer_size = 33554432
	
	
	########log settings########
	log_error = /data/3306/error.log
	slow_query_log = 1
	slow_query_log_file = /data/3306/slow.log
	log_queries_not_using_indexes = 1
	log_slow_admin_statements = 1
	log_slow_slave_statements = 1
	log_throttle_queries_not_using_indexes = 10
	expire_logs_days = 90
	long_query_time = 1
	min_examined_row_limit = 100
	
	
	
	########replication settings########
	master_info_repository = TABLE
	relay_log_info_repository = TABLE
	log_bin = /data/3306/bin.log
	sync_binlog = 1
	gtid_mode = on
	enforce_gtid_consistency = 1
	log_slave_updates
	binlog_format = row 
	relay_log = /data/3306/relay.log
	relay_log_recovery = 1
	binlog_gtid_simple_recovery = 1
	slave_skip_errors = ddl_exist_errors
	
	########innodb settings########
	##innodb_page_size = 8192
	#innodb_buffer_pool_size = 128M
	innodb_buffer_pool_instances = 8
	innodb_buffer_pool_load_at_startup = 1
	innodb_buffer_pool_dump_at_shutdown = 1
	innodb_lru_scan_depth = 2000
	innodb_lock_wait_timeout = 5
	innodb_io_capacity = 4000
	innodb_io_capacity_max = 8000
	innodb_flush_method = O_DIRECT
	innodb_file_format = Barracuda
	innodb_file_format_max = Barracuda
	#innodb_log_group_home_dir = /data/redolog/3306/
	#innodb_undo_directory = /data/undolog/3306/
	#innodb_undo_logs = 128
	innodb_undo_tablespaces = 3
	innodb_flush_neighbors = 1
	#innodb_log_file_size = 4G
	#innodb_log_buffer_size = 16777216
	innodb_purge_threads = 4
	innodb_large_prefix = 1
	innodb_thread_concurrency = 64
	innodb_print_all_deadlocks = 1
	innodb_strict_mode = 1
	#innodb_sort_buffer_size = 67108864 

victor2的配置如下：

	
	[mysqld]
	
	########basic settings########
	server-id = 1234567 
	port = 3306
	user = mysql
	
	autocommit = 1
	character_set_server=utf8mb4
	#skip_name_resolve = 1
	max_connections = 10
	max_connect_errors = 3
	datadir = /data/3306
	
	explicit_defaults_for_timestamp = 1
	#join_buffer_size = 134217728
	#tmp_table_size = 67108864
	tmpdir = /tmp
	max_allowed_packet = 16777216
	sql_mode = "STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,NO_ZERO_DATE,NO_ZERO_IN_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER"
	interactive_timeout = 1800
	wait_timeout = 1800
	#read_buffer_size = 16777216
	#read_rnd_buffer_size = 33554432
	#sort_buffer_size = 33554432
	
	
	########log settings########
	log_error = /data/3306/error.log
	slow_query_log = 1
	slow_query_log_file = /data/3306/slow.log
	log_queries_not_using_indexes = 1
	log_slow_admin_statements = 1
	log_slow_slave_statements = 1
	log_throttle_queries_not_using_indexes = 10
	expire_logs_days = 90
	long_query_time = 1
	min_examined_row_limit = 100
	
	
	
	########replication settings########
	master_info_repository = TABLE
	relay_log_info_repository = TABLE
	log_bin = /data/3306/bin.log
	sync_binlog = 1
	gtid_mode = on
	enforce_gtid_consistency = 1
	log_slave_updates
	binlog_format = row 
	relay_log = /data/3306/relay.log
	relay_log_recovery = 1
	binlog_gtid_simple_recovery = 1
	slave_skip_errors = ddl_exist_errors
	
	########innodb settings########
	##innodb_page_size = 8192
	#innodb_buffer_pool_size = 128M
	innodb_buffer_pool_instances = 8
	innodb_buffer_pool_load_at_startup = 1
	innodb_buffer_pool_dump_at_shutdown = 1
	innodb_lru_scan_depth = 2000
	innodb_lock_wait_timeout = 5
	innodb_io_capacity = 4000
	innodb_io_capacity_max = 8000
	innodb_flush_method = O_DIRECT
	innodb_file_format = Barracuda
	innodb_file_format_max = Barracuda
	#innodb_log_group_home_dir = /data/redolog/3306/
	#innodb_undo_directory = /data/undolog/3306/
	#innodb_undo_logs = 128
	innodb_undo_tablespaces = 3
	innodb_flush_neighbors = 1
	#innodb_log_file_size = 4G
	#innodb_log_buffer_size = 16777216
	innodb_purge_threads = 4
	innodb_large_prefix = 1
	innodb_thread_concurrency = 64
	innodb_print_all_deadlocks = 1
	innodb_strict_mode = 1
	#innodb_sort_buffer_size = 67108864 

需要关注的几点：

1. binlog要开
2. server-id要不一样


在主db（victor2）上`show master status`。

	mysql root@localhost:(none)> show master status\G
	***************************[ 1. row ]***************************
	File              | bin.000011
	Position          | 827
	Binlog_Do_DB      | 
	Binlog_Ignore_DB  | 
	Executed_Gtid_Set | bc5c6bda-9cf3-11e9-a250-52540002a4c4:1-10
	1 row in set
	Time: 0.008s
	mysql root@localhost:(none)>
	
记住`File`，`Position`这两个的值。

同时需要在主db上创建同步用的账户。

`create user 'repl'@'%' identifiled by '199233'`

对`repl`账户授权。

`grant replication slave on *.* to 'repl'@'%'`

然后再`victor1`的从db上，执行

	CHANGE MASTER TO
	MASTER_HOST='10.0.0.2',
	MASTER_USER='repl',
	MASTER_PASSWORD='199233',
	MASTER_LOG_FILE='bin.000011',
	MASTER_LOG_POS=827;

然后执行

	start slave;
	
通过`show slave status`查看主备复制是否ok。

	***************************[ 1. row ]***************************                                             
	Slave_IO_State                | Waiting for master to send event                                             
	Master_Host                   | 10.0.0.2
	Master_User                   | repl
	Master_Port                   | 3306
	Connect_Retry                 | 60
	Master_Log_File               | bin.000011
	Read_Master_Log_Pos           | 827
	Relay_Log_File                | relay.000002
	Relay_Log_Pos                 | 308
	Relay_Master_Log_File         | bin.000011
	Slave_IO_Running              | Yes
	Slave_SQL_Running             | Yes
	Replicate_Do_DB               |
	Replicate_Ignore_DB           |
	Replicate_Do_Table            |
	Replicate_Ignore_Table        |
	Replicate_Wild_Do_Table       |
	Replicate_Wild_Ignore_Table   |
	Last_Errno                    | 0
	Last_Error                    |
	Skip_Counter                  | 0
	Exec_Master_Log_Pos           | 827
	Relay_Log_Space               | 502
	Until_Condition               | None
	Until_Log_File                |
	Until_Log_Pos                 | 0
	Master_SSL_Allowed            | No
	Master_SSL_CA_File            |
	Master_SSL_CA_Path            |
	Master_SSL_Cert               |
	Master_SSL_Cipher             |
	Master_SSL_Key                |
	Seconds_Behind_Master         | 0
	Master_SSL_Verify_Server_Cert | No
	Last_IO_Errno                 | 0
	Last_IO_Error                 |
	Last_SQL_Errno                | 0
	Last_SQL_Error                |
	Replicate_Ignore_Server_Ids   |
	Master_Server_Id              | 1234567
	Master_UUID                   | bc5c6bda-9cf3-11e9-a250-52540002a4c4                                         
	Master_Info_File              | mysql.slave_master_info                                                      
	SQL_Delay                     | 0
	SQL_Remaining_Delay           | <null>
	Slave_SQL_Running_State       | Slave has read all relay log; waiting for the slave I/O thread to update it  
	Master_Retry_Count            | 86400
	Master_Bind                   |
	Last_IO_Error_Timestamp       |
	Last_SQL_Error_Timestamp      |
	Master_SSL_Crl                |
	Master_SSL_Crlpath            |
	Retrieved_Gtid_Set            |
	Executed_Gtid_Set             |
	Auto_Position                 | 0
	
	
可以看到io线程（Slave\_IO\_Running）和sql线程（Slave\_SQL\_Running）都ok了，说明主备复制成功建立了。

通过`show processlist`也可以看到sql线程和io线程。

	***************************[ 1. row ]***************************                                             
	Id      | 3
	User    | system user
	Host    |
	db      | <null>
	Command | Connect
	Time    | 2884
	State   | Waiting for master to send event
	Info    | <null>
	***************************[ 2. row ]***************************                                             
	Id      | 4
	User    | system user
	Host    |
	db      | <null>
	Command | Connect
	Time    | 2884
	State   | Slave has read all relay log; waiting for the slave I/O thread to update it                        
	Info    | <null>
	***************************[ 3. row ]***************************                                             
	Id      | 7
	User    | root
	Host    | localhost:32944
	db      | <null>

上面的Id为3的线程即是IO线程，Id为4的即是sql线程。

## show slave status ##

下面解释下`show slave status`的展示信息的含义。

- Slave\_IO\_State
	这里显示了当前slave I/O线程的状态(slave连接到master的状态).
	
	比如上面例子是`Waiting for master to send event `，表明slave已经连接到master，正等待它发送二进制日志。
	
	如果master闲置时，这个状态可能会持续较长时间。
- Master\_Host， Master\_User， Master\_Port

	这三个表示master db的相关信息。
	
- Connect_Retry

	连接中断后，重新尝试连接的时间间隔。默认值是60秒。
	
- Master\_Log\_File

	当前I/O线程正在读取的主服务器二进制日志文件的名称。
	
	
- Read\_Master\_Log\_Pos

	当前I/O线程正在读取的主db二进制日志的位置。
	
	
- Relay\_Log\_File

	当前slave SQL线程正在读取并执行的relay log的文件名。
	
		[root@victor1 3306]# ls
		auto.cnf    bin.000003  ib_buffer_pool  ib_logfile1         relay.000001  slow.log  undo002  victor1.pid
		bin.000001  bin.index   ibdata1         mysql               relay.000002  test      undo003
		bin.000002  error.log   ib_logfile0     performance_schema  relay.index   undo001   victor

- Relay\_Log\_Pos

	当前slave SQL线程正在读取并执行的relay log文件中的位置。（Relay\_Log\_File下的Relay\_Log\_Pos其实一一对应着Relay\_Master\_Log\_File的Exec\_Master\_Log\_Pos。）
	
- Relay\_Master\_Log\_File

	当前slave SQL线程读取并执行的relay log的文件中多数近期事件，对应的主服务器二进制日志文件的名称。(说白点就是我SQL线程从relay日志中读取的正在执行的sql语句，对应主库的sql语句记录在主库的哪个binlog日志中)
	
- Slave\_IO\_Running

	I/O线程是否被启动并成功地连接到主服务器上。
	
- Slave\_SQL\_Running

	SQL线程是否被启动。
	
- Replicate\_Do\_DB，Replicate\_Ignore\_DB，Replicate\_Do\_Table，Replicate\_Ignore\_Table，Replicate\_Wild\_Do\_Table，Replicate\_Wild\_Ignore\_Table

	这几个是用于设置复制过滤的。mysql 5.6似乎还不能直接通过`CHANGE REPLICATION FILTER ……`来设置。需要确认下。
	
- Last\_Errno ，Last\_Error

	slave的SQL线程读取日志参数的的错误数量和错误消息。
	
	错误数量为0并且消息为空字符串表示没有错误。

- Skip\_Counter

	SQL\_SLAVE\_SKIP\_COUNTER的值，用于设置跳过sql执行步数。
	
	SQL\_SLAVE\_SKIP\_COUNTER以event为单位skip，直到skip完第N个event所在的event group才停止。
	
	> This statement skips the next N events from the master. This is useful for recovering from replication stops caused by a statement.
	
	> When using this statement, it is important to understand that the binary log is actually organized as a sequence of groups known as event groups. Each event group consists of a sequence of events.
	
	> - For transactional tables, an event group corresponds to a transaction.
	> - For nontransactional tables, an event group corresponds to a single SQL statement.
When you use SET GLOBAL sql_slave_skip_counter to skip events and the result is in the middle of a group, the slave continues to skip events until it reaches the end of the group. Execution then starts with the next event group.

	binlog是按照event group组织的.
	
	对于事务表，event group 对于一个事务。
	
	对于非事务表，event group对应一个SQL。
	
	主库上的begin..commit之间对非事务表的操作记录为多个事务，每一条SQL语句对应一个event group。
	
	主库显式的在一个事务中操作事务表+非事务表，实际上所有对事务表的操作是在同一个显式事务中；所有对非事务表的操作，每条SQL语句单独对应一个事务
	
	
- Exec\_Master\_Log\_Pos

	slave SQL线程当前执行的事件，对应在master相应的二进制日志中的position。	
	
	结合Relay\_Master\_Log\_File理解，在Relay\_Master\_Log\_File这个值等于Master\_Log\_File值的时候，Exec\_Master\_Log\_Pos是不可能超过Read\_Master\_Log\_Pos的。
	
## 同一个事务中操作事务表和非事务表

	
## start slave sql\_thread