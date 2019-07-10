# mysql主备复制 #



## sql-bench ##

sql-bench是 mysql自带做基准测试的工具，在mysql目录下有个sql-bench子目录，里面存放着测试程序，这些程序都是perl实现的。

	[root@victor1 sql-bench]# pwd
	/usr/local/mysql/sql-bench
	[root@victor1 sql-bench]# ls
	bench-count-distinct  Data                   innotest2   run-all-tests     test-connect       test-wisconsin
	bench-init.pl         graph-compare-results  innotest2a  server-cfg        test-create
	compare-results       innotest1              innotest2b  test-alter-table  test-insert
	copy-db               innotest1a             limits      test-ATIS         test-select
	crash-me              innotest1b             README      test-big-tables   test-transactions
	[root@victor1 sql-bench]# 


## 表空间 ##

	[root@victor2 3306]# ls
	auto.cnf    bin.000004  bin.000008  bin.000012      ibdata1      performance_schema  undo002
	bin.000001  bin.000005  bin.000009  bin.index       ib_logfile0  slow.log            undo003
	bin.000002  bin.000006  bin.000010  error.log       ib_logfile1  test                victor
	bin.000003  bin.000007  bin.000011  ib_buffer_pool  mysql        undo001             victor2.pid
	[root@victor2 3306]# 
	
`ibdata1`就是共享表空间文件。

ibdata1是InnoDB的共有表空间，默认情况下会把表空间存放在一个文件ibdata1中，会造成这个文件越来越大。

发现问题所在之后，解决方法就是，使用独享表空间，将表空间分别单独存放。MySQL开启独享表空间的参数是Innodb_file_per_table，会为每个Innodb表创建一个.ibd的文件。

	[root@victor2 victor]# ls
	db.opt  repl_myisam.frm  repl_myisam.MYD  repl_myisam.MYI  student.frm  student.ibd
	[root@victor2 victor]# 

`student`是一张innodb表，`repl_myisam`是一张myisam表。mysql开启了独立表空间。

对于myisam表而言，`*.frm`描述了表的结构，`*.myd`保存了表的数据记录，`*.myi`则是表的索引。

对于innodb表而言，`*.frm`描述了表的结构，`*.ibd`

另外，当选项 innodb_file_per_table = 0 时，在ibdata1文件中还需要存储 InnoDB 表数据&索引。


即使使用独立表空间，共享表空间仍然存储其它的 InnoDB 内部数据，

- 数据字典，也就是 InnoDB 表的元数据
- 变更缓冲区
- 双写缓冲区
- 撤销日志

MySQL 5.6 版中可以创建外部的撤销表空间，所以它们可以放到自己的文件来替代存储到 ibdata1。

在MySQL5.6中开始支持把undo log分离到独立的表空间，并放到单独的文件目录下。


`innodb_undo_tablespaces`变量控制undolog表空间的个数，则在undo目录下创建这么多个undo文件。

在mysql_install_db时初始化后，就再也不能被改动了。

例如假定设置该值为3，那么就会创建命名为undo001~undo003的undo table space文件，每个文件的默认大小为10M。修改该值会导致Innodb无法完成初始化，数据库无法启动。

	mysql> show variables like '%undo%';                                                                         
	+-------------------------+-------+
	| Variable_name           | Value |
	+-------------------------+-------+
	| innodb_undo_directory   | .     |
	| innodb_undo_logs        | 128   |
	| innodb_undo_tablespaces | 3     |
	+-------------------------+-------+
	3 rows in set (0.00 sec)
	
	mysql>

`innodb_undo_directory `用于设置undo日志的存放目录。

如果`innodb_undo_tablespaces`为0，表示不独立设置undolog的表空间，就是放在ibdata1中。


MySQL中有六种日志文件，

1. undo
2. redo
3. bin
4. general
5. err
6. slow


`ib_logfile0`和`ib_logfile1`是redo日志。



## redolog，undolog和binlog ##

MySQL在更新数据的时候，都是将数据先从磁盘拉到buffer pool中，在buffer pool中修改完成后再写到磁盘中，也就是说MySQL中数据的更改都是要经过buffer pool的。

回到这个更新数据的过程中来看：当数据在buffer pool中更改完成的这一刻，更新后的数据是“最新”的，因为此时磁盘中的数据还是更改前的“旧数据”，而我们都是将磁盘中已经持久化的数据作为“标准数据”，因此此时buffer pool中的“最新”数据也常人们被称为“脏数据（dirty data）”。

比如将update一百行记录作为一个事务，在这个事务执行过程中会将更新后的数据先写入redo log buffer，记住是边执行边记录物理叶的修改情况。

redo log buffer再将数据刷入（请注意刷入这个用语，而非写入，后面会详细介绍）redo log中（这点和binlog不同，binlog是在事务commit后一次性写入（？？？？），而redo log在事务执行过程中就会写入）。



事务提交，但是事务中对表数据的更改不一定落盘。

redolog 防止在发生故障的时间点，尚有脏页未写入磁盘，在重启mysql服务的时候，根据redo log进行重做，从而达到事务的持久性这一特性。

`innodb_flush_log_at_trx_commit`，设置为1，表示每次事务的redolog都直接持久化到磁盘（注意是这里指的是redolog日志本身落盘），保证mysql重启后数据不丢失。

也就是说，事务提交了，就算数据没有落盘，但是肯定保证了redolog关于这次事务的更新记录落盘了。


首先，重做日志是在InnoDB存储引擎层产生，binlog日志是在数据库层面产生，binlog日志文件包括所有存储引擎对数据库更改产生的记录；

其次，两种日志记录的内容形式不同，binlog是一种逻辑日志，记录的是对应的SQL语句；而重做日志是物理格式的日志，记录的是对每一页的修改；

此外，两种日志写入磁盘的时间点不一致，bing日志只在事务提交完成以后写入；重做日志在事务提交的过程当中被不断写入；

最后，binlog文件一个事务对应一条日志记录，重做日志记录的是物理页的修改，每个事务
对应多个日志条目



- `innodb_log_file_size`：指定每个redo日志大小，默认值48MB
- `innodb_log_files_in_group`：指定日志文件组中redo日志文件数量，默认为2
- `innodb_log_group_home_dir`：指定日志文件组所在路劲，默认值./，指mysql的数据目录datadir
- `innodb_log_buffer_size`：重做日志缓冲的大小



事务提交成功，binlog就会有么？binlog有buffer没？


redo log 也是有buffer的，redo log buffer 持有将要写入relog log的数据。事务开始后，执行过程中，就会有redolog，不过这时候的redolog 是写到buffer里的。


- `innodb_flush_log_at_trx_commit	 = 1`， 用来控制redo log 刷新到磁盘的策略。默认值是1，表示每次事务提交的时候都调用fsync来写入到磁盘。这里实际上有多层含义，事务提交，是先将redo log buffer里面的内容刷到relog file里，但刷到relog file里是指刷到relog file的文件系统的缓存里。这时候如果操作系统崩溃了，relay log 还是有可能丢失。所以，还需要同时调用fsync将relog file刷到磁盘里。此时，对表的操作还没落盘，会有脏数据的情况。就是说内存中的数据和磁盘文件的不一致。
- `innodb_flush_log_at_trx_commit	 = 0`，事务执行过程中，日志一直放在redo log buffer中，但是事务commit的时候，不写入redo log file，而是通过master 线程一秒钟操作一次，从redo log buffer写入到redo log file（注意这里是同时调用fsync写入到磁盘）。
- `innodb_flush_log_at_trx_commit	 = 2`，事务提交时候，redo log buffer刷入redo log file，也就是刷入文件系统缓存，不进行fsync操作，由操作系统来做（另一种说法是，每隔一秒调用一次fsync，将文件系统缓存刷入到磁盘，MySQL 5.6.6以后，这个“1秒”的刷新还可以用innodb_flush_log_at_timeout 来控制刷新间隔。）。此时如果数据库层宕机，则不会丢失redo log，但是如果服务器宕机，这个时候文件系统中的缓存还没有fsync到磁盘文件中，这个时候就会丢失这一部分数据

当重做日志缓冲池剩余空间小于1/2时，重做日志缓冲刷新到重做日志文件（操作系统可能还有缓存）。

- `sync_binlog`，控制数据库的binlog刷到磁盘上去。`sync_binlog = 0`，表示MySQL不控制binlog的刷新，由文件系统自己控制它的缓存的刷新。这时候的性能是最好的，但是风险也是最大的。啥时候刷新呢？`sync_binlog=1`了，表示每次事务提交，MySQL都会把binlog刷下去，是最安全但是性能损耗最大的设置。


## binlog ##

事务提交成功，就会产生binlog。但是binlog不一定落到磁盘里，这个需要通过`sync_binlog `变量控制。

如果`sync_binlog`为1，那么表示每次事务提交，都会将binlog刷到磁盘里。

是刷到磁盘才返回事务提交成功？

还是立马返回成功？

binlog 的默认是保持时间由参数 expire_logs_days 配置，也就是说对于非活动的日志文件，在生成时间超过配置的天数之后，会被自动删除。	


考虑到主备同步，什么时候binlog可以被传给备机呢？是落盘后，还是写到操作系统文件缓存就可以了？

sync_binlog为1的时候，落盘才会传。

sync_binlog不为1的时候，写到操作系统文件缓存，就开始传binlog到备机了。




可以使用`show binlog events`查看binlog里面的event。

	mysql root@localhost:(none)> show master logs;
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
	| bin.000011 | 2617      |
	| bin.000012 | 756       |
	+------------+-----------+
	12 rows in set
	Time: 0.013s
	mysql root@localhost:(none)> show binlog events in 'bin.000011'\G


##mysql pid文件##

未指定 pid 文件时，pid 文件默认名为 主机名.pid，存放的路径在默认 MySQL 的数据目录。

通过 mysqld_safe 启动 MySQL 时，mysqld_safe 会检查 pid 文件，如果 pid 文件不存在，不做处理；如果文件存在，且 pid 已占用则报错 "A mysqld process already exists"，如果文件存在，但 pid 未占用，则删除 pid 文件。

查看 MySQL 的源码可以知道，mysqld 启动后会通过 create_pid_file 函数新建 pid 文件，通过 getpid() 获取当前进程 pid 并将 pid 写入 pid 文件。


因此，通过 mysqld_safe 启动时， MySQL pid 文件的作用是：在数据文件是同一份，但端口不同的情况下，防止同一个数据库被启动多次。

## GTID
从mysql 5.6开始，复制有两种实现方式，基于binlog和基于GTID。

GTID(Global Transaction ID)是对于一个已提交事务的编号，并且是一个全局唯一的编号。

GTID实际上是由UUID+TID组成的。其中UUID是一个MySQL实例的唯一标识，保存在mysql数据目录下的auto.cnf文件里。

TID代表了该实例上已经提交的事务数量，并且随着事务提交单调递增。

比如：

	[root@victor2 3306]# ls -l auto.cnf 
	-rw-rw---- 1 mysql mysql 56 7月   3 02:03 auto.cnf
	[root@victor2 3306]# cat auto.cnf 
	[auto]
	server-uuid=bc5c6bda-9cf3-11e9-a250-52540002a4c4
	[root@victor2 3306]# pwd
	/data/3306
	[root@victor2 3306]# ls
	auto.cnf    bin.000004  bin.000008  bin.000012      ibdata1      performance_schema  undo002
	bin.000001  bin.000005  bin.000009  bin.index       ib_logfile0  slow.log            undo003
	bin.000002  bin.000006  bin.000010  error.log       ib_logfile1  test                victor
	bin.000003  bin.000007  bin.000011  ib_buffer_pool  mysql        undo001             victor2.pid
	[root@victor2 3306]# 

可以看到server-uuid的值为`bc5c6bda-9cf3-11e9-a250-52540002a4c4`。

根据GTID可以知道事务最初是在哪个实例上提交的。

GTID的存在方便了Replication的Failover

GTID 模式实例和非GTID模式实例是不能进行复制的，要求非常严格，要么都是GTID，要么都不是。



开启gtid需要启用这三个参数：

	gtid_mode = on
	enforce_gtid_consistency = 1
	log_slave_updates   = 1
	
任意一个参数不开启则都会报错：

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



接下来分析下几种常见的event，其他的event类型可以参见官方文档。v4 event数据结构如下（v4: Used in MySQL 5.0 and up ）：

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
	
	
还是以上面hexdump的结果为例，第一行为：

	0000000  fe 62 69 6e 4c 9c 1b 5d  0f 87 d6 12 00 74 00 00  |.binL..].....t..|

前四个字节是魔法数字，先不管。

从第5个字节开始，往后数4个字节，即是`4c 9c 1b 5d`，这个是第一个event的时间戳。如何确认呢？

我们利用mysqlbinlog解析一下binlog文件，得到

      1 /*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
      2 /*!40019 SET @@session.max_insert_delayed_threads=0*/;
      3 /*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
      4 DELIMITER /*!*/;
      5 # at 4
      6 #190703  2:02:52 server id 1234567  end_log_pos 120 CRC32 0xe4a7fb9e    Start: binlog v 4, server v 5.
      6 6.44-log created 190703  2:02:52 at startup
      ……
      
可以看到第一个event的时间为`190703  2:02:52`，也就是`2019-07-03 02:02:52`，对应的unix时间戳为1562090572，转换为16进制为`5d1b9c4c`，刚好是上面hexdump出来的第5-8字节。这里也可以看到binlog是小端字节序。


binlog文件的第一个event是`FORMAT_DESCRIPTION_EVENT`。

event的类型就是通过上面的`type_code`来确定。

	enum Log_event_type { 
	  UNKNOWN_EVENT= 0, 
	  START_EVENT_V3= 1, 
	  QUERY_EVENT= 2, 
	  STOP_EVENT= 3, 
	  ROTATE_EVENT= 4, 
	  INTVAR_EVENT= 5, 
	  LOAD_EVENT= 6, 
	  SLAVE_EVENT= 7, 
	  CREATE_FILE_EVENT= 8, 
	  APPEND_BLOCK_EVENT= 9, 
	  EXEC_LOAD_EVENT= 10, 
	  DELETE_FILE_EVENT= 11, 
	  NEW_LOAD_EVENT= 12, 
	  RAND_EVENT= 13, 
	  USER_VAR_EVENT= 14, 
	  FORMAT_DESCRIPTION_EVENT= 15, 
	  XID_EVENT= 16, 
	  BEGIN_LOAD_QUERY_EVENT= 17, 
	  EXECUTE_LOAD_QUERY_EVENT= 18, 
	  TABLE_MAP_EVENT = 19, 
	  PRE_GA_WRITE_ROWS_EVENT = 20, 
	  PRE_GA_UPDATE_ROWS_EVENT = 21, 
	  PRE_GA_DELETE_ROWS_EVENT = 22, 
	  WRITE_ROWS_EVENT = 23, 
	  UPDATE_ROWS_EVENT = 24, 
	  DELETE_ROWS_EVENT = 25, 
	  INCIDENT_EVENT= 26, 
	  HEARTBEAT_LOG_EVENT= 27, 
	  IGNORABLE_LOG_EVENT= 28,
	  ROWS_QUERY_LOG_EVENT= 29,
	  WRITE_ROWS_EVENT = 30,
	  UPDATE_ROWS_EVENT = 31,
	  DELETE_ROWS_EVENT = 32,
	  GTID_LOG_EVENT= 33,
	  ANONYMOUS_GTID_LOG_EVENT= 34,
	  PREVIOUS_GTIDS_LOG_EVENT= 35, 
	  ENUM_END_EVENT 
	  /* end marker */ 
	};

可以看到`FORMAT_DESCRIPTION_EVENT `的`type_code`取值为15。

接下来是`87 d6 12 00`，表示server-id，server-id的值为`12345678`。

接下来`74 00 00 00`，表示event的长度，值为116。加上binlog的魔法数字4个字节，所以第一个event的`end_log_pos`的值为120，与mysqlbinlog解析出来的值也一样。

后面就不细说了。

>A format description event is the first event of a binlog for binlog-version 4. It describes how the other events are layed out. 

## flush logs ##

`flush logs`会切分当前的binlog文件，得到一个新的binlog文件。会生成一个rotate event。
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