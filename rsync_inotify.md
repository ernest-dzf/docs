# 利用 rsync 和 inotify 同步两个机器的文件#
我们有时候会有这个需求，需要将A机器某个目录下的文件实时同步到B机器某个目录下。可以利用rsync和inotify来解决。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/rsync_inotify.png)

<<<<<<< Updated upstream
## rsync ##
rsync是一个远程数据同步工具。

rsync使用所谓的 “Rsync 算法” 来使本地和远程两个主机之间的文件达到同步，这个算法只传送两个文件的不同部分，而不是每次都整份传送，因此速度相当快。


rsync可以使用rcp、ssh等方式来传输文件，当然也可以通过直接的socket连接。

在使用 rsync 进行远程同步时，可以使用两种方式：远程 Shell 方式（建议使用 ssh，用户验证由 ssh 负责）和 C/S 方式（即客户连接远程 rsync 服务器，用户验证由 rsync 服务器负责）。

无论本地同步目录还是远程同步数据，首次运行时将会把全部文件拷贝一次，以后再运行时将只拷贝有变化的文件（对于新文件）或文件的变化部分（对于原有文件）。

### rsync配置文件 ###
rsync最重要的就是配置文件。先来看一下官方给的一个例子，如下：



> A more sophisticated example would be:


	uid = nobody
	gid = nobody
	use chroot = yes
	max connections = 4
	syslog facility = local5
	pid file = /var/run/rsyncd.pid
	
	[ftp]
	        path = /var/ftp/./pub
	        comment = whole ftp area (approx 6.1 GB)
	
	[sambaftp]
	        path = /var/ftp/./pub/samba
	        comment = Samba ftp area (approx 300 MB)
	
	[rsyncftp]
	        path = /var/ftp/./pub/rsync
	        comment = rsync ftp area (approx 6 MB)
	
	[sambawww]
	        path = /public_html/samba
	        comment = Samba WWW pages (approx 240 MB)
	
	[cvs]
	        path = /data/cvs
	        comment = CVS repository (requires authentication)
	        auth users = tridge, susan
	        secrets file = /etc/rsyncd.secrets

> The /etc/rsyncd.secrets file would look something like this:

	tridge:mypass
	susan:herpass

该文件是由一个或多个模块结构组成。一个模块定义是以方括弧中的模块名开始，直到下一个模块定义开始或者文件结束。

文件是由一个或多个模块结构组成。一个模块定义是以方括弧中的模块名开始，直到下一个模块定义开始或者文件结束。

模块中包含格式为name = value的参数定义。每个模块其实就对应需要备份的一个目录树，比方说在我们的实际环境中，有三个目录树需要备份：/www/、/home/web_user1/和/home/web_user2/，那么就需要在配置文件中定义三个模块，分别对应三个目录树。


## inotify ##
## 脚本 ##

在发起端机器B上需要运行一个脚本，如下：

	# victordong @ VM_144_188_centos in /data/victor/rsynctest [23:00:35] 
	$ cat rsync.sh 
	#!/bin/bash 
	host=10.104.126.59
	src=/data/victor/rsynctest
	des=tdsql
	user=root
	/usr/bin/inotifywait -mrq --timefmt '%d/%m/%y %H:%M' --format '%T %w%f%e' -e modify,delete,create,attrib $src | while read files 
	do
	        /usr/bin/rsync -vzrtopg --delete --progress --password-file=/data/victor/.rsync/pwd $src $user@$host::$des >/dev/null 2>&1
	        echo "${files} was rsynced" >>/tmp/rsync.log 2>&1 
	done
	
	# victordong @ VM_144_188_centos in /data/victor/rsynctest [23:00:39] 
	$ 

rsync.sh的作用就是利用inotify实时监控文件的变化，并在文件有被更改的时候，利用rsync将变动同步到服务器A。
=======
## inotify安装 ##
1. 下载，https://sourceforge.net/projects/inotify-tools/
2. 安装

		./configure && make && make install

	可能有权限问题，sudo一下或者直接用root。

	安装好后，会有这两个东西inotifywait和inotifywatch，默认安装目录是`/usr/local/bin`

		# victor @ localhost in ~ [22:11:37] 
		$ ls /usr/local/bin/inotify*   
		/usr/local/bin/inotifywait  /usr/local/bin/inotifywatch
		
		# victor @ localhost in ~ [22:13:55] 
		$ 
### rsync安装 ###

rsync一般默认已经安装，如果没有的话，用yum安装一下。
>>>>>>> Stashed changes
