# ssh可以连接，sftp不能用 #

出问题的一般是.bashrc的配置。

如果sftp提示连接超时，可能.bashrc中有某个命令特别耗时或者，该放到后台的命令没有放在后台。

例如，我是用zsh，不用bash，但是服务器没有安装zsh，我是一个只有家目录权限的用户，所以在/home/xxx/.local/zsh安装了zsh，chsh不能使用/bin之外的，只好把zsh加在了配置文件中。

	PATH=$HOME/.local/zsh/bin:$PATH
	zsh


问题就出现了，sftp利用ssh通道的时候发现bash怎么也请求不完，一直卡在zsh，就出现了超时的问题。