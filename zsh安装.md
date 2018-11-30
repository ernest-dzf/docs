# zsh安装 #
这里以centos 7为例。

1. yum install zsh。
2. 修改账户的默认shell。

		usermod -s /bin/zsh victor
3. 重新以victor账户登录，查看是否更改成功。

		# victor @ localhost in ~ [9:26:06] C:1
		$ echo $0
		-zsh
		

	可以看到，当前使用的shell已经是zsh。

	主目录下也有`.zshrc`文件，类似于`.bashrc`文件。

#oh-my-zsh安装#


oh-my-zsh 帮我们整理了一些常用的 Zsh 扩展功能和主题。

zsh配合oh-my-zsh使用更佳。


1. `yum install git`
2. `sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`
3. 修改主题

		……
		# See https://github.com/robbyrussell/oh-my-zsh/wiki/Themes
		ZSH_THEME="ys"
		
		# Set list of themes to pick from when loading at random
		……
4. 退出重新登陆即可

上面为victor用户安装了oh-my-zsh，如果需要root用户也可以用到的话，需要修改root用户的默认shell，方法也是利用usermod命令。

	usermod -s /bin/zsh root

然后，将victorz主目录下面的`.oh-my-zsh`和`.zshrc`拷贝到root主目录下面去。注意修改`.zshrc`中的	ZSH变量。

	……
	# Path to your oh-my-zsh installation.
	  export ZSH="/root/.oh-my-zsh"
	
	# Set name of the theme to load --- if set to "random", it will
	……

修改为root用户的.oh-my-zsh所在的目录。

然后重新登录root用户，就可以了。