# mysql配置文件详解 #

先贴一份配置文件，里面包含了我们比较常用的配置项。

	[client]
	user=wind
	password=199233
	port=3306
	
	[mysqld]
	########basic settings########
	server-id = 11 
	port = 3306
	user = mysql
	bind_address = 10.166.224.32
	autocommit = 0
	character_set_server=utf8mb4
	skip_name_resolve = 1
	max_connections = 800
	max_connect_errors = 1000
	datadir = /data/mysql_data
	transaction_isolation = READ-COMMITTED
	explicit_defaults_for_timestamp = 1
	join_buffer_size = 134217728
	tmp_table_size = 67108864
	tmpdir = /tmp
	##--skip-networking
	##read_only=1
	max_allowed_packet = 16777216
	sql_mode = "STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,NO_ZERO_DATE,NO_ZERO_IN_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER"
	interactive_timeout = 1800
	wait_timeout = 1800
	read_buffer_size = 16777216
	read_rnd_buffer_size = 33554432
	sort_buffer_size = 33554432
	########log settings########
	log_error = error.log
	slow_query_log = 1
	slow_query_log_file = slow.log
	log_queries_not_using_indexes = 1
	log_slow_admin_statements = 1
	log_slow_slave_statements = 1
	log_throttle_queries_not_using_indexes = 10
	expire_logs_days = 90
	long_query_time = 2
	min_examined_row_limit = 100
	########replication settings########
	master_info_repository = TABLE
	relay_log_info_repository = TABLE
	log_bin = bin.log
	sync_binlog = 1
	gtid_mode = on
	enforce_gtid_consistency = 1
	log_slave_updates
	binlog_format = row 
	relay_log = relay.log
	relay_log_recovery = 1
	binlog_gtid_simple_recovery = 1
	slave_skip_errors = ddl_exist_errors
	########innodb settings########
	innodb_page_size = 8192
	innodb_buffer_pool_size = 6G
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
	innodb_log_group_home_dir = /redolog/
	innodb_undo_directory = /undolog/
	innodb_undo_logs = 128
	innodb_undo_tablespaces = 3
	innodb_flush_neighbors = 1
	innodb_log_file_size = 4G
	innodb_log_buffer_size = 16777216
	innodb_purge_threads = 4
	innodb_large_prefix = 1
	innodb_thread_concurrency = 64
	innodb_print_all_deadlocks = 1
	innodb_strict_mode = 1
	innodb_sort_buffer_size = 67108864 
	########semi sync replication settings########
	plugin_dir=/usr/local/mysql/lib/plugin
	plugin_load = "rpl_semi_sync_master=semisync_master.so;rpl_semi_sync_slave=semisync_slave.so"
	loose_rpl_semi_sync_master_enabled = 1
	loose_rpl_semi_sync_slave_enabled = 1
	loose_rpl_semi_sync_master_timeout = 5000
	
	[mysqld-5.7]
	innodb_buffer_pool_dump_pct = 40
	innodb_page_cleaners = 4
	innodb_undo_log_truncate = 1
	innodb_max_undo_log_size = 2G
	innodb_purge_rseg_truncate_frequency = 128
	binlog_gtid_simple_recovery=1
	log_timestamps=system
	transaction_write_set_extraction=MURMUR32
	show_compatibility_56=on


## client配置

	[client]
	user=wind
	password=199233
	port=3306
	
这几行配置表示在mysqld服务器上，使用`mysql`登录时，默认使用的账户和密码。

比如下面：
	
	[root@VM_0_15_centos data]# mysql
	Welcome to the MySQL monitor.  Commands end with ; or \g.
	Your MySQL connection id is 13
	Server version: 5.6.43-log MySQL Community Server (GPL)
	
	Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
	
	Oracle is a registered trademark of Oracle Corporation and/or its
	affiliates. Other names may be trademarks of their respective
	owners.
	
	Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
	
	mysql> select user();
	+----------------+
	| user()         |
	+----------------+
	| wind@localhost |
	+----------------+
	1 row in set (0.00 sec)
	
	mysql> quit
	Bye
	[root@VM_0_15_centos data]# 
	
我们使用`mysql`命令登录，发现登录使用的用户名就是`mysql.cnf`配置文件中`[client]`段指出的用户。

这样我们可以不用输入一堆命令行就可以登录mysql了。

这种操作一般就在本机做。


## server-id ##

server-id用于唯一地标识一个数据库实例。在配置文件中指定该mysql实例的server-id。

这个一般用在mysql的主从复制当中。务必保证不会出现重复。

简单来说，server-id有三个用途：

1. 标记binlog event的来源地，即SQL语句最开始源自哪里。
2. mysql做主主同步时，多个主需要构成一个环，但是同步的时候需要保证一条数据不会陷入死循环，这里就是靠server-id来实现的。
3. 每一个同步中的slave在master上都对应一个master线程，该线程就是通过slave的server-id来标识的。每个slave在master端最多有一个master线程，如果两个slave的server-id 相同，则后一个连接成功时，前一个将被踢掉。这里至少有这么一种考虑，slave主动连接master之后，如果slave上面执行了`slave stop`，则连接断开，但是master上对应的线程并没有退出。当`slave start`之后，master不能再创建一个线程，同时保留原来的线程，那样同步就可能有问题。

## --skip-networking ##

如果配置文件中有这一行，那么mysql就不会接受tcp连接。

## bind_address ##

如果在配置文件中指定`bind_address`为`127.0.0.1`的话，那么mysql将只接受本机过来的连接请求。

也就是说，你无法远程连接mysql。

`bind_address`默认取值为`*`。

>If the address is *, the server accepts TCP/IP connections on all server host IPv4 interfaces, and, if the server host supports IPv6, on all IPv6 interfaces. Use this address to permit both IPv4 and IPv6 connections on all server interfaces. This value is the default. 
>

## read_only=1 ##

设置db只读。

可以在配置文件中固化，或者通过`set global read_only=1`来设置。

不过`set global`之后，重启就会变回默认的。

## max_connections ##

MySQL的最大连接数，增加该值增加 mysqld 要求的文件描述符的数量。

如果服务器的并发连接请求量比较大，建议调高此值，以增加并行连接数量，当然这建立在机器能支撑的情况下，因为如果连接数越多，鉴于MySQL会为每个连接提供连接缓冲区，就会开销越多的内存，所以要适当调整该值，不能盲目提高设值。

mysql无论如何都会保留一个用于管理员（SUPER）登陆的连接，用于管理员连接数据库进行维护操作，即使当前连接数已经达到了max_connections。因此MySQL的实际最大可连接数为max_connections+1。超过的话，再连接就会报错。

	[root@victor2 ~]# mysql -uroot
	ERROR 1040 (HY000): Too many connections
	[root@victor2 ~]# 

可以这样查看当前使用的连接数。

	mysql> show status like '%max_used%';
	+----------------------+-------+
	| Variable_name        | Value |
	+----------------------+-------+
	| Max_used_connections | 2     |
	+----------------------+-------+
	1 row in set (0.00 sec)
	
	mysql>

## character\_set\_server ##

设置字符集。具体参看另外一篇介绍。

## max\_connect\_errors ##

最大连接错误数。

如果mysql服务器连续接收到了来自于同一个主机的请求，且这些连续的请求全部都没有成功的建立连接就被断开了，当这些连续的请求的累计值大于 max_connect_errors的设定值时，mysql服务器就会阻止这台主机后续的所有请求。

这个变量和密码错误次数无关。

TODO:需要进一步查找资料。