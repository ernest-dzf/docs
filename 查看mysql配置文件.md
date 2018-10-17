# mysql 查看当前使用的配置文件 #
my.cnf是mysql启动时加载的配置文件，一般会放在mysql的安装目录中，用户也可以放在其他目录加载。

安装mysql后，系统中会有多个my.cnf文件，有些是用于测试的。

使用locate my.cnf命令可以列出所有的my.cnf文件。

当然也有可能指定了某个配置文件，它的名字不叫`my.cnf`，这时候可以`ps`一下，看下启动进程信息。比如下面这样：

	# victor @ localhost in ~ [4:48:52] 
	$ ps -ef | grep mysql
	root       1379      1  0 02:03 ?        00:00:00 /bin/sh /usr/local/mysql/bin/mysqld_safe --datadir=/usr/local/mysql/data --pid-file=/usr/local/mysql/data/localhost.localdomain.pid
	mysql      1548   1379  0 02:03 ?        00:00:06 /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=localhost.localdomain.err --pid-file=/usr/local/mysql/data/localhost.localdomain.pid
	victor     1969   1927  0 02:53 pts/1    00:00:00 mysql -uroot -p
	
	# victor @ localhost in ~ [4:48:54] 
	$ 


如果需要查看所有的my.cnf文件，可以这样，

	# victor @ localhost in ~ [4:48:54] 
	$ locate my.cnf
	/usr/local/Cellar/mysql/5.6.24/my.cnf
	/usr/local/Cellar/mysql/5.6.24/mysql-test/include/default_my.cnf
	/usr/local/Cellar/mysql/5.6.24/mysql-test/suite/federated/my.cnf
	/usr/local/Cellar/mysql/5.6.24/mysql-test/suite/ndb/my.cnf
	/usr/local/Cellar/mysql/5.6.24/mysql-test/suite/ndb_big/my.cnf
	/usr/local/Cellar/mysql/5.6.24/mysql-test/suite/ndb_binlog/my.cnf
	/usr/local/Cellar/mysql/5.6.24/mysql-test/suite/ndb_rpl/my.cnf
	/usr/local/Cellar/mysql/5.6.24/mysql-test/suite/ndb_team/my.cnf
	/usr/local/Cellar/mysql/5.6.24/mysql-test/suite/rpl/extension/bhs/my.cnf
	/usr/local/Cellar/mysql/5.6.24/mysql-test/suite/rpl/my.cnf
	/usr/local/Cellar/mysql/5.6.24/mysql-test/suite/rpl_ndb/my.cnf

当我们需要修改配置文件时，需要找到mysql启动时是加载了哪个my.cnf文件。

## 查看是否使用了指定目录的配置文件 ##

	# victor @ localhost in ~ [4:51:17] 
	$ ps -ef | grep mysql
	root       1379      1  0 02:03 ?        00:00:00 /bin/sh /usr/local/mysql/bin/mysqld_safe --datadir=/usr/local/mysql/data --pid-file=/usr/local/mysql/data/localhost.localdomain.pid
	mysql      1548   1379  0 02:03 ?        00:00:06 /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=localhost.localdomain.err --pid-file=/usr/local/mysql/data/localhost.localdomain.pid
	victor     1969   1927  0 02:53 pts/1    00:00:00 mysql -uroot -p
	
	# victor @ localhost in ~ [4:52:31] 
	$ 

上面可以看到，并没有使用指定的配置文件

## 查看mysql默认读取配置文件的目录 ##

	# victor @ localhost in ~/mysql [4:55:02] 
	$ mysql --help|grep 'my.cnf'
	                      order of preference, my.cnf, $MYSQL_TCP_PORT,
	/etc/my.cnf /etc/mysql/my.cnf /usr/local/mysql/etc/my.cnf ~/.my.cnf 
	
	# victor @ localhost in ~/mysql [4:55:15] 
	$ 


这些就是mysql默认会搜寻my.cnf的目录，顺序排前的优先。

## 启动时没有使用配置文件 ##

如果没有设置使用指定目录配置文件及默认读取目录没有my.cnf文件，表示mysql启动时并没有加载配置文件，而是使用默认配置。

需要修改配置，可以在mysql默认读取的目录中，创建一个my.cnf文件(例如:/etc/my.cnf)，把需要修改的配置内容写入，重启mysql后即可生效。