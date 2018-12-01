# vmware 虚拟机 命令行安装vmware-tools #
1. 在vmware控制台上给虚拟机安装vmware-tools，这实际上是将vmware-tools iso文件挂载到虚拟机上去。

	![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/vmware-tools.png)
2. 挂载vmware-tools的iso文件

		mount -t iso9660 /dev/cdrom /mnt/cdrom

3. 可以看到`/mnt/cdrom`目录下有如下文件

		# root @ localhost in /mnt/cdrom [12:56:36] 
		$ ls
		manifest.txt  run_upgrader.sh  VMwareTools-10.1.15-6627299.tar.gz  vmware-tools-upgrader-32  vmware-tools-upgrader-64
		
		# root @ localhost in /mnt/cdrom [12:56:37] 
		$ 
	
	其中`VMwareTools-10.1.15-6627299.tar.gz`就是我们需要的，将其拷贝到你的主目录。
		
		# root @ localhost in /mnt/cdrom [12:56:37] 
		$ cp VMwareTools-10.1.15-6627299.tar.gz ~/
		
		# root @ localhost in /mnt/cdrom [12:57:38] 
		$ cd ~
		
		# root @ localhost in ~ [12:57:40] 
		$ ls -l
		总用量 55060
		-rw-------. 1 root root     1423 12月  1 08:31 anaconda-ks.cfg
		-r--r--r--. 1 root root 56375699 12月  1 12:57 VMwareTools-10.1.15-6627299.tar.gz
		drwxr-xr-x. 9 root root      145 9月  14 2017 vmware-tools-distrib
		
		# root @ localhost in ~ [12:57:41] 
		$ 

	然后解压`VMwareTools-10.1.15-6627299.tar.gz`文件。进入到`vmware-tools-distrib`目录。

		# root @ localhost in ~/vmware-tools-distrib [12:58:19] 
		$ ls
		bin  caf  doc  etc  FILES  INSTALL  installer  lib  vgauth  vmware-install.pl
		
		# root @ localhost in ~/vmware-tools-distrib [12:58:44] 
		$ ./vmware-install.pl 

	执行`vmware-install.pl`。

	安装好之后，就可以在vmware控制台上设置共享目录了。