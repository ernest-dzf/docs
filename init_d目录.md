# /etc/init.d目录 #
如果你使用过linux系统，那么你一定听说过init.d目录。这个目录到底是干嘛的呢？

init.d目录包含许多系统各种服务的启动和停止脚本。它控制着从acpid到x11-common的各种事务。当然，init.d远远没有这么简单。（acpid 是linux操作系统新型电源管理标准；X11也叫做X Window系统，X Window系统 (X11或X)是一种位图显示的视窗系统 。它是在 Unix 和 类Unix 操作系统 ，以及 OpenVMS 上建立图形用户界面 的标准工具包和协议，并可用于几乎已有的现代操作系统）。


当你查看`/etc`目录时，你会发现许多rc#.d 形式存在的目录（这里#代表一个指定的初始化级别，范围是0~6）。在这些目录之下，包含了许多对进程进行控制的脚本。这些脚本要么以"K"开头，要么以"S"开头。以K开头的脚本运行在以S开头的脚本之前。这些脚本放置的地方，将决定这些脚本什么时候开始运行。然而，有时候你希望能在不使用kill 或kill all 命令的情况下，能干净的启动或杀死一个进程。这就是/etc/init.d能够派上用场的地方了！

为了能够使用init.d目录下的脚本，你需要有root权限或sudo权限。每个脚本都将被作为一个命令运行，该命令的结构大致如下所示：

	/etc/init.d/command 选项

comand是实际运行的命令，选项可以有如下几种：

- start
- stop
- reload
- restart
- force-reload

大多数的情况下，你会使用start,stop,restart选项。例如，如果你想关闭网络，你可以使用如下形式的命令：

	/etc/init.d/networking stop

又比如，你改变了网络设置，并且需要重启网络。你可以使用如下命令：

	/etc/init.d/networking restart

或者查看mysql服务运行状态：

	# root @ localhost in /etc/init.d [8:58:25] 
	$ service mysqld status
	 SUCCESS! MySQL running (3067)
	
	# root @ localhost in /etc/init.d [8:59:02] 
	$ 
