# yum常用命令 #
## yum install ##
安装包，安装某一个包可以这样，`yum install package1`
## yum info ##
显示某一个包的信息，比如这样，

	$ yum info rsync
	Loaded plugins: fastestmirror
	Loading mirror speeds from cached hostfile
	 * base: mirrors.aliyun.com
	 * extras: mirrors.aliyun.com
	 * updates: centos.ustc.edu.cn
	Installed Packages
	Name        : rsync
	Arch        : x86_64
	Version     : 3.1.2
	Release     : 4.el7
	Size        : 815 k
	Repo        : installed
	From repo   : base
	Summary     : A program for synchronizing files over a network
	URL         : http://rsync.samba.org/
	License     : GPLv3+
	Description : Rsync uses a reliable algorithm to bring remote and host files into
	            : sync very quickly. Rsync is fast because it just sends the differences
	            : in the files over the network instead of sending the complete
	            : files. Rsync is often used as a very powerful mirroring process or
	            : just as a more capable replacement for the rcp command. A technical
	            : report which describes the rsync algorithm is included in this
	            : package.
	
	
	# victor @ localhost in ~ [22:56:58] 
	$ 

## yum search ##
按照某个关键字搜索一个包，比如这样

	# victor @ localhost in ~ [22:56:58] 
	$ yum search rsync
	Loaded plugins: fastestmirror
	Loading mirror speeds from cached hostfile
	 * base: mirrors.aliyun.com
	 * extras: mirrors.aliyun.com
	 * updates: centos.ustc.edu.cn
	================================================================== N/S matched: rsync ===================================================================
	libguestfs-rsync.x86_64 : rsync support for libguestfs
	rsync.x86_64 : A program for synchronizing files over a network
	
	  Name and summary matches only, use "search all" for everything.
	
	# victor @ localhost in ~ [22:57:19] 
	$ 
## yum list ##

显示指定包安装情况，比如这样

	$ yum list gcc
	Loaded plugins: fastestmirror
	Loading mirror speeds from cached hostfile
	 * base: mirrors.aliyun.com
	 * extras: mirrors.aliyun.com
	 * updates: centos.ustc.edu.cn
	Installed Packages
	gcc.x86_64                                                           4.8.5-28.el7_5.1                                                            @updates
	
	# victor @ localhost in ~ [22:59:09] 
	$ 
## yum remove ##

删除程序包，比如这样

	$ sudo yum remove gcc
	Loaded plugins: fastestmirror
	Resolving Dependencies
	--> Running transaction check
	---> Package gcc.x86_64 0:4.8.5-28.el7_5.1 will be erased
	--> Finished Dependency Resolution
	
	Dependencies Resolved
	
	=========================================================================================================================================================
	 Package                        Arch                              Version                                      Repository                           Size
	=========================================================================================================================================================
	Removing:
	 gcc                            x86_64                            4.8.5-28.el7_5.1                             @updates                             37 M
	
	Transaction Summary
	=========================================================================================================================================================
	Remove  1 Package
	
	Installed size: 37 M
	Is this ok [y/N]: 
## yum deplist ##

获取一个包的依赖情况信息，比如这样

	# victor @ localhost in ~ [23:01:48] 
	$ yum deplist tmux
	Loaded plugins: fastestmirror
	Loading mirror speeds from cached hostfile
	 * base: mirrors.aliyun.com
	 * extras: mirrors.aliyun.com
	 * updates: centos.ustc.edu.cn
	package: tmux.x86_64 1.8-4.el7
	  dependency: /bin/sh
	   provider: bash.x86_64 4.2.46-30.el7
	  dependency: libc.so.6(GLIBC_2.14)(64bit)
	   provider: glibc.x86_64 2.17-222.el7
	  dependency: libevent-2.0.so.5()(64bit)
	   provider: libevent.x86_64 2.0.21-4.el7
	  dependency: libncurses.so.5()(64bit)
	   provider: ncurses-libs.x86_64 5.9-14.20130511.el7_4
	  dependency: libresolv.so.2()(64bit)
	   provider: glibc.x86_64 2.17-222.el7
	  dependency: libresolv.so.2(GLIBC_2.2.5)(64bit)
	   provider: glibc.x86_64 2.17-222.el7
	  dependency: libtinfo.so.5()(64bit)
	   provider: ncurses-libs.x86_64 5.9-14.20130511.el7_4
	  dependency: libutil.so.1()(64bit)
	   provider: glibc.x86_64 2.17-222.el7
	  dependency: libutil.so.1(GLIBC_2.2.5)(64bit)
	   provider: glibc.x86_64 2.17-222.el7
	  dependency: rtld(GNU_HASH)
	   provider: glibc.x86_64 2.17-222.el7
	   provider: glibc.i686 2.17-222.el7
## yum repolist all ##

类似的，`yum repolist enabled`，`yum repolist disabled`

	# victor @ localhost in /etc/yum.repos.d [0:52:09] 
	$ yum repolist enabled 
	repo id                                                                 repo name                                                                  status
	base/7/x86_64                                                           CentOS-7 - Base                                                            9,911
	extras/7/x86_64                                                         CentOS-7 - Extras                                                            432
	updates/7/x86_64                                                        CentOS-7 - Updates                                                         1,602
	repolist: 11,945
	
	# victor @ localhost in /etc/yum.repos.d [0:52:27] 
	$ 

## yum 源 ##

和yum源相关的主要2个地方，如下：

	# victor @ localhost in /etc/yum.repos.d [0:52:27] 
	$ pwd
	/etc/yum.repos.d
	
	# victor @ localhost in /etc/yum.repos.d [0:53:18] 
	$ cat /etc/yum.conf 
	[main]
	cachedir=/var/cache/yum/$basearch/$releasever
	keepcache=0
	debuglevel=2
	logfile=/var/log/yum.log
	exactarch=1
	obsoletes=1
	gpgcheck=1
	plugins=0
	installonly_limit=5
	bugtracker_url=http://bugs.centos.org/set_project.php?project_id=23&ref=http://bugs.centos.org/bug_report_page.php?category=yum
	distroverpkg=centos-release
	
	
	#  This is the default, if you make this bigger yum won't see if the metadata
	# is newer on the remote and so you'll "gain" the bandwidth of not having to
	# download the new metadata and "pay" for it by yum not having correct
	# information.
	#  It is esp. important, to have correct metadata, for distributions like
	# Fedora which don't keep old packages around. If you don't like this checking
	# interupting your command line usage, it's much better to have something
	# manually check the metadata once an hour (yum-updatesd will do this).
	# metadata_expire=90m
	
	# PUT YOUR REPOS HERE OR IN separate files named file.repo
	# in /etc/yum.repos.d
	
	# victor @ localhost in /etc/yum.repos.d [0:53:23] 
	$ 


也就是`/etc/yum.repos.d`目录和`/etc/yum.conf`文件。

`/etc/yum.repos.d`目录下有一些XXXX.repo文件，比如下面：

	# victor @ localhost in /etc/yum.repos.d [0:53:23] 
	$ ls
	CentOS-Base.repo  CentOS-CR.repo  CentOS-Debuginfo.repo  CentOS-fasttrack.repo  CentOS-Media.repo  CentOS-Sources.repo  CentOS-Vault.repo
	
	# victor @ localhost in /etc/yum.repos.d [0:55:06] 
	$ 

通常一个repo文件定义了一个或者多个软件仓库的细节内容，例如我们将从哪里下载需要安装或者升级的软件包，repo文件中的设置内容将被YUM读取和应用。

YUM的基本工作流程如下：

服务器端：在服务器上面存放了所有的RPM软件包，然后以相关的功能去分析每个RPM文件的依赖性关系，将这些数据记录成文件存放在服务器的某特定目录内。

客户端：如果需要安装某个软件时，先下载服务器上面记录的依赖性关系文件(可通过WWW或FTP方式)，通过对服务器端下载的纪录数据进行分析，然后取得所有相关的软件，一次全部下载下来进行安装。

下面给出一个范例的repo文件

	# victor @ localhost in /etc/yum.repos.d [1:07:33] 
	$ cat CentOS-Base.repo 
	# CentOS-Base.repo
	#
	# The mirror system uses the connecting IP address of the client and the
	# update status of each mirror to pick mirrors that are updated to and
	# geographically close to the client.  You should use this for CentOS updates
	# unless you are manually picking other mirrors.
	#
	# If the mirrorlist= does not work for you, as a fall back you can try the 
	# remarked out baseurl= line instead.
	#
	#
	
	[base]
	name=CentOS-$releasever - Base
	mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
	#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
	gpgcheck=1
	gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
	
	#released updates 
	[updates]
	name=CentOS-$releasever - Updates
	mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
	#baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
	gpgcheck=1
	gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
	
	#additional packages that may be useful
	[extras]
	name=CentOS-$releasever - Extras
	mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
	#baseurl=http://mirror.centos.org/centos/$releasever/extras/$basearch/
	gpgcheck=1
	gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
	
	#additional packages that extend functionality of existing packages
	[centosplus]
	name=CentOS-$releasever - Plus
	mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
	#baseurl=http://mirror.centos.org/centos/$releasever/centosplus/$basearch/
	gpgcheck=1
	enabled=0
	gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7



以上面为例解释各个字段的含义。

- [base]

	表示各个不同的repository，必须有一个独一无二的名称
- name=CentOS-$releasever - Base

	表示对repository的描述。
- mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra

	指定镜像服务器的地址列表。我们可以尝试，将$releasever和$basearch替换成自己对应的版本和架构，例如7和x86_64，在浏览器中打开，我们就能看到一长串可用的镜像服务器地址列表。centos从中选择一个链接来下载更新。
	
	或者我们手动选择自己访问速度较快的镜像服务器地址复制并粘贴到repo文件中的baseurl选项中。

	![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/yum_mirrorlist.png)
	
- baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/

	实际上就是某个仓库，比如下面，我们将$releasever和$basearch替换成对应的值后，放到浏览器访问，可以看到就是一个仓库。

	![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/repo.png)
- gpgcheck=1

	有1和0两个选择，分别代表是否是否进行gpg校验，如果没有这一项，默认是检查的。
- gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
	`file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7`表示秘钥文件，公钥。用户下载rpm包时，使用gpgkey验证RPM包是不是RH官方签名的。

## yum -y makecache ##

服务器的包信息下载到本地电脑缓存起来，配合yum -C search xxx使用不用上网检索就能查找软件信息

## yum provides ifconfig ##

搜索哪个包提供`ifconfig`这个命令

	# root @ localhost in ~ [9:00:23] C:1
	$ yum provides ifconfig
	Loaded plugins: fastestmirror
	Loading mirror speeds from cached hostfile
	 * base: mirrors.cn99.com
	 * extras: mirrors.aliyun.com
	 * updates: mirrors.aliyun.com
	base                                                                                                                              | 3.6 kB  00:00:00     
	extras                                                                                                                            | 3.4 kB  00:00:00     
	updates                                                                                                                           | 3.4 kB  00:00:00     
	(1/2): extras/7/x86_64/primary_db                                                                                                 | 205 kB  00:00:00     
	(2/2): updates/7/x86_64/primary_db                                                                                                | 6.0 MB  00:00:01     
	extras/7/x86_64/filelists_db                                                                                                      | 603 kB  00:00:00     
	updates/7/x86_64/filelists_db                                                                                                     | 3.4 MB  00:00:00     
	net-tools-2.0-0.22.20131004git.el7.x86_64 : Basic networking tools
	Repo        : @base
	Matched from:
	Filename    : /usr/sbin/ifconfig
	
比如上面，包`net-tools`提供`ifconfig`这个命令。