# centos 7 安装完无法连接网络问题 #


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

# centos 7 安装完没有ifconfig命令问题 #
	
安装net-tools包。

	yum install net-tools