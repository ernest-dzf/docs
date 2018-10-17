# mysql auto.cnf文件 #
## auto.cnf文件 ##
MySQL数据目录中通常存在一个名为auto.cnf文件，存储了server-uuid的值，如下所示：

	[auto]
	server-uuid=778b50dd-cc1d-11e8-8b03-000c299cf654

MySQL启动时，会自动从data_dir/auto.cnf 文件中获取server-uuid值，并将这个值存储在全局变量server_uuid中。如果这个值或者这个文件不存在，那么将会生成一个新的uuid值，并将这个值保存在auto.cnf文件中。

	# root @ localhost in /usr/local/mysql/data [3:01:37] 
	$ pwd
	/usr/local/mysql/data
	
	# root @ localhost in /usr/local/mysql/data [3:02:04] 
	$ ls
	auto.cnf    client-cert.pem  ibdata1      ibtmp1                     mysql               public_key.pem   sys
	ca-key.pem  client-key.pem   ib_logfile0  localhost.localdomain.err  performance_schema  server-cert.pem  victor
	ca.pem      ib_buffer_pool   ib_logfile1  localhost.localdomain.pid  private_key.pem     server-key.pem

比如上面的**auto.cnf文件**。

server-uuid与server-id一样，用于标识MySQL实例在集群中的唯一性，这两个参数在主从复制中具有重要作用，默认情况下，如果主、从库的server-uuid或者server-id的值一样，将会导致主从复制报错中断。

在主库中执行show slave hosts; 能够看到主、从库的server-id和从库的server-uuid，如下所示：

	mysql> show slave hosts;
	+-----------+------+------+-----------+--------------------------------------+
	| Server_id | Host | Port | Master_id | Slave_UUID                           |
	+-----------+------+------+-----------+--------------------------------------+
	| 170238777 |      | 3306 | 170238776 | 5a0280d4-6404-11e8-840e-fa163eab3dcf |
	| 170238778 |      | 3306 | 170238776 | 4db07c8a-6f01-11e8-bef6-fa163e767b9a |
	+-----------+------+------+-----------+--------------------------------------+
	2 rows in set (0.00 sec)


在从库中执行show slave status\G，能够看到主库的server-id和server-uuid，如下所示：

							......
	         Master_Server_Id: 170238776
	              Master_UUID: f2d0efd6-6ab7-11e8-8fdd-fa163eda7360
	         Master_Info_File: mysql.slave_master_info
	                SQL_Delay: 0
	      SQL_Remaining_Delay: NULL
	  Slave_SQL_Running_State:
	       Master_Retry_Count: 86400
	                       ......


## 主、从库server-uuid值一样，导致主从复制异常中断 ##

MySQL在初始化的时候，会去数据目录里的auto.cnf文件读取server-uuid参数，如果没有auto.cnf文件或者没有读到server-uuid参数，通过执行generate_server_uuid 函数生成一个新的uuid值，并通过flush_auto_options函数将server-uuid值写入到auto.cnf文件中。具体的函数调用及源码文件如下：

	main                       //main.cc
	mysqld_main                //mysqld.cc
	init_server_auto_options   //mysqld.cc
	generate_server_uuid       //mysqld.cc
	flush_auto_options         //mysqld.cc

主从复制时，如果从库发现它所复制的主库的uuid 与 它自己的uuid一样，默认情况下从库的I/O线程会直接报错退出。

MySQL提供了额外参数--replicate-same-server-id，用于改变这一默认行为。如果设置replicate-same-server-id为1，即使主从库uuid一样，也能正常复制，但是会产生意想不到的结果，实际使用中也很少会这么使用。

相关代码实现位于rpl_slave.cc文件的get_master_uuid函数中，具体源码实现如下：

	main                //main.cc
	mysqld_main         //mysqld.cc
	mysqld_socket_acceptor->connection_event_loop    //mysqld.cc
	Connection_handler_manager::process_new_connection(Channel_info* channel_info)  //connection_handler_manager.cc
	bool Per_thread_connection_handler::add_connection(Channel_info* channel_info)  //connection_handler_per_thread.cc
	handle_connection        //connection_handler_per_thread.cc
	do_command               //sql_parse.cc
	dispatch_command         //sql_parse.cc
	mysql_parse              //sql_parse.cc
	mysql_execute_command    //sql_parse.cc
	start_slave_cmd          //rpl_slave.cc
	start_slave              //rpl_slave.cc
	start_slave_threads      //rpl_slave.cc
	handle_slave_io          //rpl_slave.cc
	get_master_uuid          //rpl_slave.cc

，

	//rpl_slave.cc
	//static int get_master_uuid(MYSQL *mysql, Master_info *mi)
	
	char query_buf[]= "SELECT @@GLOBAL.SERVER_UUID";
	...
	if (!mysql_real_query(mysql, STRING_WITH_LEN(query_buf)) &&
	      (master_res= mysql_store_result(mysql)) &&
	      (master_row= mysql_fetch_row(master_res)))
	  {
	    if (!strcmp(::server_uuid, master_row[0]) &&
	        !mi->rli->replicate_same_server_id)
	    {
	      errmsg= "The slave I/O thread stops because master and slave have  equal "
	              "MySQL server UUIDs; these UUIDs must be different for "
	              "replication to work.";
	      mi->report(ERROR_LEVEL, ER_SLAVE_FATAL_ERROR,  ER(ER_SLAVE_FATAL_ERROR),
	                 errmsg);
	      // Fatal error
	      ret= 1;
	    }
	
## 主、从库server-id值一样，同样导致主从复制异常中断 ##

如果主从复制时，从库发现它所复制的主库的server_id 与 它自己的server_id一样，默认情况下从库的I/O线程也会直接报错退出。

通过设置--replicate-same-server-id 参数为1，可改变这一默认行为。

下面是执行start slave;命令时的函数调用关系。

	main                  //main.cc
	mysqld_main           //mysqld.cc
	mysqld_socket_acceptor->connection_event_loop    //mysqld.cc
	Connection_handler_manager::process_new_connection(Channel_info* channel_info)    //connection_handler_manager.cc
	bool Per_thread_connection_handler::add_connection(Channel_info* channel_info)    //connection_handler_per_thread.cc
	handle_connection                   //connection_handler_per_thread.cc
	do_command                          //sql_parse.cc
	dispatch_command                    //sql_parse.cc
	mysql_parse                         //sql_parse.cc
	mysql_execute_command               //sql_parse.cc
	start_slave_cmd                     //rpl_slave.cc
	start_slave                         //rpl_slave.cc
	start_slave_threads                 //rpl_slave.cc
	handle_slave_io                     //rpl_slave.cc
	get_master_version_and_clock        //rpl_slave.cc
	

下面是MySQL从库启动时，如果设置了skip-slave-start，从库自动启动复制线程的函数调用关系。

	main                                  //main.cc
	mysqld_main                     //mysqld.cc
	init_slave                            //rpl_slave.cc
	start_slave_threads            //rpl_slave.cc
	handle_slave_io                 //rpl_slave.cc
	get_master_version_and_clock    //rpl_slave.cc


最终都是通过调用get_master_version_and_clock函数，来判断主、从库server-id是否相同。

	// rpl_slave.cc
	// static int get_master_version_and_clock(MYSQL* mysql, Master_info* mi)
	
	if (!mysql_real_query(mysql, STRING_WITH_LEN("SELECT @@GLOBAL.SERVER_ID")) &&
	      (master_res= mysql_store_result(mysql)) &&
	      (master_row= mysql_fetch_row(master_res)))
	  {
	    if ((::server_id == (mi->master_id= strtoul(master_row[0], 0, 10))) &&
	        !mi->rli->replicate_same_server_id)
	    {
	      errmsg= "The slave I/O thread stops because master and slave have equal \
	MySQL server ids; these ids must be different for replication to work (or \
	the --replicate-same-server-id option must be used on slave but this does \
	not always make sense; please check the manual before using it).";
	      err_code= ER_SLAVE_FATAL_ERROR;
	      sprintf(err_buff, ER(err_code), errmsg);
	      goto err;
	    }
	  }

