# ssh连接慢 #

ssh连接远程主机时，有时候会出现这种情况，并卡在这儿：
	
	Connecting to 192.168.1.2:22...
	Connection established.
	To escape to local shell, press 'Ctrl+Alt+]'.

查了文档，会出现这种问题是因为ssh默认有一个配置项UseDNS，引用官方文档说法是：

> UseDNS Specifies whether sshd should look up the remote host name and check that the resolved host name for the remote IP address maps back to the very same IP address. The default is “yes”.

UseDNS默认值为yes，这个选项打开的状态下，当客户端试图登录SSH服务器时，服务器端先根据客户端的IP地址进行DNS反向查询出客户端的主机名，然后根据查询出的客户端主机名进行DNS正向记录查询，验证与其原始IP地址是否一致，所以在登陆的时候会出现卡顿。可以通过重设这个选项的值，改为no就可以解决。

	……
	#UseLogin no
	#UsePrivilegeSeparation sandbox
	#PermitUserEnvironment no
	#Compression delayed
	#ClientAliveInterval 0
	#ClientAliveCountMax 3
	#ShowPatchLevel no
	UseDNS no
	#PidFile /var/run/sshd.pid
	#MaxStartups 10:30:100
	#PermitTunnel no
	#ChrootDirectory none
	#VersionAddendum none
	……

然后重启ssh服务，问题解决

	# victor @ localhost in ~ [18:47:02] 
	$ service sshd restart
