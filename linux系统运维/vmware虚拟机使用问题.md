[toc]
# vmware虚拟机使用问题
## centos 7 安装完无法连接网络问题 


	[victor@localhost]~% cat  /etc/sysconfig/network-scripts/ifcfg-ens33
	TYPE=Ethernet
	PROXY_METHOD=none
	BROWSER_ONLY=no
	BOOTPROTO=dhcp
	DEFROUTE=yes
	IPV4_FAILURE_FATAL=no
	IPV6INIT=yes
	IPV6_AUTOCONF=yes
	IPV6_DEFROUTE=yes
	IPV6_FAILURE_FATAL=no
	IPV6_ADDR_GEN_MODE=stable-privacy
	NAME=ens33
	UUID=57a08b6c-a050-4ffc-b64f-b4603b6fa57d
	DEVICE=ens33
	ONBOOT=yes
	[victor@localhost]~% 

修改`/etc/sysconfig/network-scripts/ifcfg-ens33`文件，将`ONBOOT`修改为yes，然后重启网络服务。


	[victor@localhost]~% service network restart 
	Restarting network (via systemctl):  ==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
	Authentication is required to manage system services or units.
	Authenticating as: root
	Password: 
	==== AUTHENTICATION COMPLETE ===
	                                                           [  确定  ]
	[victor@localhost]~% 

## centos 7 安装完没有ifconfig命令问题 
	
安装net-tools包。

	yum install net-tools
	
	
	
## vmwre虚拟机nat网络 
有时候需要虚拟机固定ip地址，然后又需要可以连上外网，这时候可以使用nat网络。
### vmware虚拟机配置 

配置如下图，重点关注网关地址，这里可以看到是192.168.111.2。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/nat%E8%AE%BE%E7%BD%AE.png)

### centos 配置

配置如下：

	# victor @ localhost in ~ [21:53:14] 
	$ cat /etc/sysconfig/network-scripts/ifcfg-ens33
	TYPE=Ethernet
	PROXY_METHOD=none
	BROWSER_ONLY=no
	BOOTPROTO=static
	DEFROUTE=yes
	IPV4_FAILURE_FATAL=no
	IPV6INIT=yes
	IPV6_AUTOCONF=yes
	IPV6_DEFROUTE=yes
	IPV6_FAILURE_FATAL=no
	IPV6_ADDR_GEN_MODE=stable-privacy
	NAME=ens33
	UUID=988d2248-6f38-4bfb-b713-f75ca52dd1ed
	DEVICE=ens33
	ONBOOT=yes
	IPADDR=192.168.111.129
	GATEWAY=192.168.111.2
	NETMASK=255.255.255.0
	DNS1=192.168.111.2
	
	# victor @ localhost in ~ [21:53:48] 
	$ 

重点关注GATEWAY和DNS1这两个字段的设置，可以看到值为192.168.111.2，与虚拟机nat设置的网关值一样。

再来看下母机上的地址，

	以太网适配器 VMware Network Adapter VMnet8:
	
	   连接特定的 DNS 后缀 . . . . . . . : 
	   本地链接 IPv6 地址. . . . . . . . : fe80::3cff:516b:7c34:9786%5
	   IPv4 地址 . . . . . . . . . . . . : 192.168.111.1
	   子网掩码  . . . . . . . . . . . . : 255.255.255.0
	   默认网关. . . . . . . . . . . . . : 

可以看到地址为192.168.111.1，而网关地址为192.168.111.2，注意区分。




## vmware 虚拟机 命令行安装vmware-tools 
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

	