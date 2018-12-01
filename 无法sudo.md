# 无法sudo的问题 #

解决用户无法sudo的问题，修改`/etc/sudoers`文件：

	……
	## The COMMANDS section may have other options added to it.
	##
	## Allow root to run any commands anywhere 
	root    ALL=(ALL)       ALL
	victor  ALL=(ALL)       ALL
	
	## Allows members of the 'sys' group to run networking, software, 
	……

在`root    ALL=(ALL)       ALL`下面添加一行`victor  ALL=(ALL)       ALL`即可。

编辑之前注意修改文件的权限，编辑完成后，再把权限改回去。

# 解释 #

`root    ALL=(ALL)       ALL`

who whereHost=(runas) commond

谁  通过哪些主机=(可以通过哪个身份)运行哪些命令

