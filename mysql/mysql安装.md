# mysql安装 #
这里以centos为例。

## percona 5.7 安装 ##

1. 官网下载，地址：[https://www.percona.com/downloads/Percona-Server-5.7/Percona-Server-5.7.25-28/binary/tarball/Percona-Server-5.7.25-28-Linux.x86_64.ssl101.tar.gz](https://www.percona.com/downloads/Percona-Server-5.7/LATEST/binary/tarball/)。
	
	这里注意下载版本，官方说法是：
> 	Percona Server offers multiple tarballs depending on the OpenSSL library available in the distribution:
 
	> ssl100 - for Debian prior to 9 and Ubuntu prior to 14.04 versions (libssl.so.1.0.0 => /usr/lib/x86_64-linux-gnu/libssl.so.1.0.0 (0x00007f2e389a5000));
	
	> ssl101 - for CentOS 6 and CentOS 7 (libssl.so.10 => /usr/lib64/libssl.so.10 (0x00007facbe8c4000));
	
	> ssl102 - for Debian 9 and Ubuntu versions starting from 14.04 (libssl.so.1.1 => /usr/lib/libssl.so.1.1 (0x00007f5e57397000);
	
	我的机器是centos 7，所以下载**ssl101**版本。

2. 得到`Percona-Server-5.7.25-28-Linux.x86_64.ssl101.tar.gz`tar包，解压到`/usr/local`目录下面去

		[root@VM_0_15_centos mysql]# tar -zxvf Percona-Server-5.7.25-28-Linux.x86_64.ssl101.tar.gz -C /usr/local/
		[root@VM_0_15_centos local]# ls
		bin    include  libexec                                       sbin   yd.socket.server
		etc    lib      Percona-Server-5.7.25-28-Linux.x86_64.ssl101  share
		games  lib64    qcloud                                        src
		[root@VM_0_15_centos local]# 
3. 创建一个软链接到`Percona-Server-5.7.25-28-Linux.x86_64.ssl101`这个目录

		[root@VM_0_15_centos local]# ls
		bin  games    lib    libexec  Percona-Server-5.7.25-28-Linux.x86_64.ssl101  sbin   src
		etc  include  lib64  mysql    qcloud                                        share  yd.socket.server
		[root@VM_0_15_centos local]# ls -l mysql
		lrwxrwxrwx 1 root root 44 4月  20 11:42 mysql -> Percona-Server-5.7.25-28-Linux.x86_64.ssl101
		[root@VM_0_15_centos local]# 


## mysql 5.7安装 ##
1. 官网下载linux generic binary包，地址：https://dev.mysql.com/downloads/mysql/
2. 下载得到的是一个xxx.tar.gz的包，比如`mysql-5.7.23-linux-glibc2.12-x86_64.tar.gz`，解压得到`/usr/local/`目录下面去，比如

		# victor @ localhost in /usr/local [8:40:55] 
		$ ls
		bin  etc  games  include  lib  lib64  libexec  mysql  mysql-5.7.23-linux-glibc2.12-x86_64  sbin  share  src
3. 在`/usr/local`目录下建立一个软链接到`mysql-5.7.23-linux-glibc2.12-x86_64`目录

		# victor @ localhost in /usr/local [8:44:20] C:1
		$ ls
		bin  etc  games  include  lib  lib64  libexec  mysql-5.7.23-linux-glibc2.12-x86_64  sbin  share  src
		
		# victor @ localhost in /usr/local [8:44:23] 
		$ sudo ln -s mysql-5.7.23-linux-glibc2.12-x86_64 mysql
		
		# victor @ localhost in /usr/local [8:44:31] 
		$ ls  
		bin  etc  games  include  lib  lib64  libexec  mysql  mysql-5.7.23-linux-glibc2.12-x86_64  sbin  share  src
		
		# victor @ localhost in /usr/local [8:44:36] 
		$ ls -l mysql
		lrwxrwxrwx. 1 root root 35 Oct 10 08:44 mysql -> mysql-5.7.23-linux-glibc2.12-x86_64
4. 安装一些依赖包

		shell> yum search libaio  # search for info
		shell> yum install libaio # install library

5. 安装指令

		shell> groupadd mysql
		shell> useradd -r -g mysql -s /bin/false mysql
		shell> cd /usr/local
		shell> tar zxvf /path/to/mysql-VERSION-OS.tar.gz
		shell> ln -s full-path-to-mysql-VERSION-OS mysql
		shell> cd mysql
		shell> mkdir mysql-files
		shell> chown mysql:mysql mysql-files
		shell> chmod 750 mysql-files
		shell> bin/mysqld --initialize --user=mysql 
		shell> bin/mysql_ssl_rsa_setup              
		shell> bin/mysqld_safe --user=mysql &
		# Next command is optional
		shell> cp support-files/mysql.server /etc/init.d/mysql.server

	上面中mysql.server脚本用于方便地启动、暂停、重启服务。比如下面这样：

		# victor @ localhost in ~ [8:10:40] C:3
		$ sudo /etc/init.d/mysql.server start
		Starting MySQL.. SUCCESS! 
		
		# victor @ localhost in ~ [8:10:50] 
		$ sudo /etc/init.d/mysql.server status
		 SUCCESS! MySQL running (2095)
		
		# victor @ localhost in ~ [8:10:53] 
		$ sudo /etc/init.d/mysql.server stop  
		Shutting down MySQL.. SUCCESS! 
		
		# victor @ localhost in ~ [8:11:00] 
		$ 

6. 添加环境变量

	在`/etc/profile`文件末尾添加如下一行

		export PATH=$PATH:/usr/local/mysql/bin
	然后 `source /etc/profile`

7. others

	安装完成后可以看到`/usr/local/mysql`目录下的布局如下

		# root @ localhost in /usr/local/mysql [8:54:53] 
		$ pwd
		/usr/local/mysql
		
		# root @ localhost in /usr/local/mysql [8:54:54] 
		$ ls
		bin  COPYING  data  docs  include  lib  man  mysql-files  README  share  support-files
		
		# root @ localhost in /usr/local/mysql [8:54:55]

	其中data目录放的就是数据

8. 如果想设置开机启动的话

	为了方便，重命名一下先：

		# root @ localhost in /etc/init.d [9:01:28] 
		$ mv /etc/init.d/mysql.server mysqld

	然后：		

		# root @ localhost in /etc/init.d [9:01:10] C:1
		$ chkconfig --add mysqld          
		
		# root @ localhost in /etc/init.d [9:01:18] 
		$ chkconfig --list      
		
		Note: This output shows SysV services only and does not include native
		      systemd services. SysV configuration data might be overridden by native
		      systemd configuration.
		
		      If you want to list systemd services use 'systemctl list-unit-files'.
		      To see services enabled on particular target use
		      'systemctl list-dependencies [target]'.
		
		mysqld         	0:off	1:off	2:on	3:on	4:on	5:on	6:off
		netconsole     	0:off	1:off	2:off	3:off	4:off	5:off	6:off
		network        	0:off	1:off	2:on	3:on	4:on	5:on	6:off
		vmware-tools   	0:off	1:off	2:on	3:on	4:on	5:on	6:off
		vmware-tools-thinprint	0:off	1:off	2:on	3:on	4:on	5:on	6:off

	可以看到mysqld在3、4、5运行级别下都是on，说明设置成功。

## mysql5.6安装 ##

	shell> groupadd mysql
	shell> useradd -r -g mysql mysql
	shell> cd /usr/local
	shell> tar zxvf /path/to/mysql-VERSION-OS.tar.gz
	shell> ln -s full-path-to-mysql-VERSION-OS mysql
	shell> cd mysql
	shell> chown -R mysql .
	shell> chgrp -R mysql .
	shell> scripts/mysql_install_db --user=mysql
	shell> chown -R root .
	shell> chown -R mysql data
	shell> bin/mysqld_safe --user=mysql &
	# Next command is optional
	shell> cp support-files/mysql.server /etc/init.d/mysql.server
	
1. 首先添加mysql用户组
2. 添加mysql账户，设置用户组为mysql
3. 在`/usr/local`目录下建立一个到`full-path-to-mysql-VERSION-OS`的软链接
4. 修改`mysql`目录的用户组属性为`mysql:mysql`
5. 执行安装`scripts/mysql_install_db --user=mysql`
6. 更改`mysql`目录及其子目录下所有目录、文件的所属账户为root，即保持`mysql`目录属性为`root:mysql`
7. 更改`data`目录的所属用户为`mysql`
8. 启动mysqld
9. 加入到service中，重新登录后就可以用`service start mysql`或者`service stop mysql`启停mysql数据库了

其他的诸如加到自启动，环境变量的配置，步骤和mysql5.7安装一样。