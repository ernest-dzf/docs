# vmwre虚拟机nat网络 #
有时候需要虚拟机固定ip地址，然后又需要可以连上外网，这时候可以使用nat网络。
## vmware虚拟机配置 ##

配置如下图，重点关注网关地址，这里可以看到是192.168.111.2。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/nat%E8%AE%BE%E7%BD%AE.png)

## centos 配置 ##

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
