# linux 运行级别 #

## 介绍 ##

 Linux 下分了多种运行级别，所谓的运行级别就是当前系统启动后能做什么！

运行级别如下：

|级别|说明|
|:--|:--|
|0|关机|
|1|单用户【可以找回丢失密码】|
|2|多用户状态但是没有网络|
|3|多用户有网络服务|
|4|系统未使用，保留给用户|
|5|图形界面|
|6|系统重启|

常见的Linux运行级别为3或5

## 运行级别原理 ##

1. 在目录`/etc/rc.d/init.d`下有许多服务器脚本程序，一般称为服务(service)，比如下面：
	
	```
	# victor @ localhost in /etc/rc.d/init.d [9:14:06] 
	$ pwd
	/etc/rc.d/init.d
		
	# victor @ localhost in /etc/rc.d/init.d [9:14:08] 
	$ ls
	functions  mysqld  netconsole  network  README  vmware-tools  vmware-tools-thinprint
		
	# victor @ localhost in /etc/rc.d/init.d [9:14:09] 
	$ 
	```
	
2. 在`/etc/rc.d`下有7个名为rcN.d的目录，对应系统的7个运行级别，比如下面：

		# victor @ localhost in /etc/rc.d [9:10:41] 
		$ pwd
		/etc/rc.d
		
		# victor @ localhost in /etc/rc.d [9:10:46] 
		$ ls
		init.d  rc0.d  rc1.d  rc2.d  rc3.d  rc4.d  rc5.d  rc6.d  rc.local
		
		# victor @ localhost in /etc/rc.d [9:10:47] 
		$ 

3. rcN.d目录下都是一些符号链接文件，这些链接文件都指向init.d目录下的service脚本文件，命名规则为K+nn+服务名或S+nn+服务名，其中nn为两位数字。 比如rc3.d目录下的文件可能如下面所示：

		# victor @ localhost in /etc/rc3.d [9:05:08] 
		$ ls -l
		total 0
		lrwxrwxrwx. 1 root root 20 Sep 23 20:18 K50netconsole -> ../init.d/netconsole
		lrwxrwxrwx. 1 root root 22 Sep 23 21:53 S03vmware-tools -> ../init.d/vmware-tools
		lrwxrwxrwx. 1 root root 17 Sep 23 20:18 S10network -> ../init.d/network
		lrwxrwxrwx. 1 root root 32 Sep 23 21:53 S57vmware-tools-thinprint -> ../init.d/vmware-tools-thinprint
		lrwxrwxrwx. 1 root root 16 Oct 11 09:01 S64mysqld -> ../init.d/mysqld

4. 系统会根据指定的运行级别进入对应的rcN.d目录，并按照文件名顺序检索目录下的链接文件：对于以K(Kill)开头的文件，系统将终止对应的服务；对于以S(Start)开头的文件，系统将启动对应的服务。
5. 查看运行级别，比如下面：

		# victor @ localhost in /etc/rc.d/init.d [9:14:09] 
		$ runlevel 
		N 3
		
		# victor @ localhost in /etc/rc.d/init.d [9:16:47] 
		$ 
6. 进入其他运行level，使用`init N`命令，N替换为运行级别