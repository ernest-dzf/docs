# shell相关 #
### 常用的shell ###

常用的shell有这么几种，可以通过`cat /etc/shells`查看。

	# victordong @ VM_144_188_centos in ~ [23:06:48] 
	$ cat /etc/shells
	/bin/sh
	/bin/bash
	/usr/bin/sh
	/usr/bin/bash
	/bin/tcsh
	/bin/csh
	/bin/ksh
	/bin/rksh
	/bin/zsh
	
	# victordong @ VM_144_188_centos in ~ [23:06:55] 
	$ 

要查看当前使用的shell，可以`echo $0`，如下：

	# victordong @ VM_144_188_centos in ~ [23:06:55] 
	$ echo $0
	/bin/zsh
	
	# victordong @ VM_144_188_centos in ~ [23:08:04] 
	$ 

使用的zsh。