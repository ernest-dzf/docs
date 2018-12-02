# mysql pid文件 #

在mysql的数据目录中，我们可能会看到一个xxxxx.pid的文件，它的作用是啥呢？

	# victor @ localhost in /usr/local/mysql/data [22:44:08] 
	$ ls
	auto.cnf  ibdata1  ib_logfile0  ib_logfile1  localhost.localdomain.err  localhost.localdomain.pid  mysql  performance_schema  test  victor
	
	# victor @ localhost in /usr/local/mysql/data [22:54:22] 
	$ 

比如上面的localhost.localdomain.pid文件。

MySQL pid 文件记录的是当前 mysqld 进程的 pid，pid 亦即 Process ID。

	# victor @ localhost in /usr/local/mysql/data [22:55:39] C:1
	$ ps -ef | grep mysqld         
	root       2065      1  0 18:53 ?        00:00:00 /bin/sh /usr/local/mysql/bin/mysqld_safe --datadir=/usr/local/mysql/data --pid-file=/usr/local/mysql/data/localhost.localdomain.pid
	mysql      2166   2065  0 18:53 ?        00:00:07 /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=/usr/local/mysql/data/localhost.localdomain.err --pid-file=/usr/local/mysql/data/localhost.localdomain.pid
	
	# victor @ localhost in /usr/local/mysql/data [22:55:43] 
	$ sudo cat localhost.localdomain.pid
	[sudo] victor 的密码：
	2166
	
	# victor @ localhost in /usr/local/mysql/data [22:55:50] 
	$ 

比如上面的例子，mysqld进场的Process Id为2166。

## 未指定 pid 文件 ##

pid 文件默认名为 主机名.pid，存放的路径在默认 MySQL 的数据目录。

通过 mysqld_safe 启动 MySQL 时，mysqld_safe 会检查 pid 文件，如果 pid 文件不存在，不做处理；如果文件存在，且 pid 已占用则报错 "A mysqld process already exists"，如果文件存在，但 pid 未占用，则删除 pid 文件。

查看 MySQL 的源码可以知道，mysqld 启动后会通过 create_pid_file 函数新建 pid 文件，通过 getpid() 获取当前进程 pid 并将 pid 写入 pid 文件。

因此，通过 mysqld_safe 启动时， MySQL pid 文件的作用是：在数据文件是同一份，但端口不同的情况下，防止同一个数据库被启动多次。