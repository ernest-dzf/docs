# zsh安装 #
这里以centos 7为例。

1. yum install zsh。
2. 修改账户的默认shell。
	```
	usermod -s /bin/zsh victor
	```
3. 重新以victor账户登录，查看是否更改成功。
	```
	# victor @ localhost in ~ [9:26:06] C:1
	$ echo $0
	-zsh
	```

	可以看到，当前使用的shell已经是zsh。

	主目录下也有`.zshrc`文件，类似于`.bashrc`文件。

#oh-my-zsh安装#


oh-my-zsh 帮我们整理了一些常用的 Zsh 扩展功能和主题。

zsh配合oh-my-zsh使用更佳。


1. `yum install git`
2. `sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`
3. 修改主题
	```
	# See https://github.com/robbyrussell/oh-my-zsh/wiki/Themes
	ZSH_THEME="ys"
	
	# Set list of themes to pick from when loading at random
	```
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

# zsh一些插件 #

## autojump ##

autojump是一个命令行工具，它允许你可以直接跳转到你喜爱的目录，而不用管你现在身在何处。

1. 安装autojump
	
	先下载：

		https://github.com/wting/autojump.git

	安装：

		# victor @ localhost in ~ [12:48:43] 
		$ ls
		autojump
		
		# victor @ localhost in ~ [12:48:44] 
		$ cd autojump 
		
		# victor @ localhost in ~/autojump on git:master o [12:48:49] 
		$ ls
		AUTHORS  bin  CHANGES.md  docs  install.py  LICENSE  Makefile  README.md  tests  tools  tox.ini  uninstall.py
		
		# victor @ localhost in ~/autojump on git:master o [12:48:50] 
		$ ./install.py --help
		usage: install.py [-h] [-n] [-f] [-d DIR] [-p DIR] [-z DIR] [-c DIR] [-s]
		
		Installs autojump globally for root users, otherwise installs in current
		user's home directory.
		
		optional arguments:
		  -h, --help            show this help message and exit
		  -n, --dryrun          simulate installation
		  -f, --force           skip root user, shell type, Python version checks
		  -d DIR, --destdir DIR
		                        set destination to DIR
		  -p DIR, --prefix DIR  set prefix to DIR
		  -z DIR, --zshshare DIR
		                        set zsh share destination to DIR
		  -c DIR, --clinkdir DIR
		                        set clink directory location to DIR (Windows only)
		  -s, --system          install system wide for all users
		
		# victor @ localhost in ~/autojump on git:master o [12:48:54] 

	执行`autojump`目录下的install.py文件即可，
		
		# victor @ localhost in ~/autojump on git:master o [14:59:06] 
		$ ls
		AUTHORS  bin  CHANGES.md  docs  install.py  LICENSE  Makefile  README.md  tests  tools  tox.ini  uninstall.py
		
		# victor @ localhost in ~/autojump on git:master o [14:59:10] 
		$ ./install.py 
		Installing autojump to /home/victor/.autojump ...
		creating directory: /home/victor/.autojump/bin
		creating directory: /home/victor/.autojump/share/man/man1
		creating directory: /home/victor/.autojump/etc/profile.d
		creating directory: /home/victor/.autojump/share/autojump
		copying file: ./bin/autojump -> /home/victor/.autojump/bin
		copying file: ./bin/autojump_argparse.py -> /home/victor/.autojump/bin
		copying file: ./bin/autojump_data.py -> /home/victor/.autojump/bin
		copying file: ./bin/autojump_match.py -> /home/victor/.autojump/bin
		copying file: ./bin/autojump_utils.py -> /home/victor/.autojump/bin
		copying file: ./bin/icon.png -> /home/victor/.autojump/share/autojump
		copying file: ./docs/autojump.1 -> /home/victor/.autojump/share/man/man1
		creating directory: /home/victor/.autojump/etc/profile.d
		creating directory: /home/victor/.autojump/share/autojump
		creating directory: /home/victor/.autojump/functions
		copying file: ./bin/autojump.sh -> /home/victor/.autojump/etc/profile.d
		copying file: ./bin/autojump.bash -> /home/victor/.autojump/share/autojump
		copying file: ./bin/autojump.fish -> /home/victor/.autojump/share/autojump
		copying file: ./bin/autojump.zsh -> /home/victor/.autojump/share/autojump
		copying file: ./bin/_j -> /home/victor/.autojump/functions
		
		Please manually add the following line(s) to ~/.zshrc:
		
			[[ -s /home/victor/.autojump/etc/profile.d/autojump.sh ]] && source /home/victor/.autojump/etc/profile.d/autojump.sh
		
			autoload -U compinit && compinit -u
		
		Please restart terminal(s) before running autojump.
	
	
	​	
	​	# victor @ localhost in ~/autojump on git:master o [14:59:12] 
	$ 
	
	安装过程有提示说，让在.zshrc文件中添加几行，其实没必要。我们只需要在plugins中设置加载autojump即可。这在后面有介绍。
	

安装完成后，还需要把autojump加入到系统PATH目录中去，在.zshrc开头加入如下一行：
	
	export ZSH="/home/victor/.oh-my-zsh"

退出重新登录账户，然后验证一下：
	
		# victor @ localhost in ~/autojump on git:master o [12:52:32] 
		$ autojump -v
		autojump v22.5.3
		
		# victor @ localhost in ~/autojump on git:master o [12:52:34] 
	$ 

如果提示找不到命令的话，需要查看下PATH路径是不是加错了。
	
2. 设置.zshrc文件

	编辑.zshrc文件，找到类似下面的几行
		

		……
		# Example format: plugins=(rails git textmate ruby lighthouse)
		# Add wisely, as too many plugins slow down shell startup.
		plugins=(
		  git autojump
		)
		
		source $ZSH/oh-my-zsh.sh
		
		# User configuration
		……
	
	在`plugins=(git)`后面添加autojump，即表示加载autojump插件。

	保存退出，重新登录账户，或者source一下.zshrc文件即可。

3. 踩到的坑

	- 没有安装autojump，就直接在.zshrc文件中plugins中设置加载autojump，这肯定不得行
	- 没有把autojump加入到系统PATH中去。

		其实看下oh-my-zsh中的代码就了解了

			# victor @ localhost in ~/autojump on git:master o [15:07:43] 
			$ cat  ~/.oh-my-zsh/plugins/autojump/autojump.plugin.zsh
			if [ $commands[autojump] ]; then # check if autojump is installed
			  if [ -f $HOME/.autojump/etc/profile.d/autojump.zsh ]; then # manual user-local installation
			    . $HOME/.autojump/etc/profile.d/autojump.zsh
			  elif [ -f $HOME/.autojump/share/autojump/autojump.zsh ]; then # another manual user-local installation
			    . $HOME/.autojump/share/autojump/autojump.zsh
			  elif [ -f $HOME/.nix-profile/etc/profile.d/autojump.zsh ]; then # nix installation
			    . $HOME/.nix-profile/etc/profile.d/autojump.zsh
			  elif [ -f /run/current-system/sw/share/autojump/autojump.zsh ]; then # nixos installation
			    . /run/current-system/sw/share/autojump/autojump.zsh
			  elif [ -f /usr/share/autojump/autojump.zsh ]; then # debian and ubuntu package
			    . /usr/share/autojump/autojump.zsh
			  elif [ -f /etc/profile.d/autojump.zsh ]; then # manual installation
			    . /etc/profile.d/autojump.zsh
			  elif [ -f /etc/profile.d/autojump.sh ]; then # gentoo installation
			    . /etc/profile.d/autojump.sh
			  elif [ -f /usr/local/share/autojump/autojump.zsh ]; then # freebsd installation
			    . /usr/local/share/autojump/autojump.zsh
			  elif [ -f /opt/local/etc/profile.d/autojump.sh ]; then # mac os x with ports
			    . /opt/local/etc/profile.d/autojump.sh
			  elif [ $commands[brew] -a -f `brew --prefix`/etc/autojump.sh ]; then # mac os x with brew
			    . `brew --prefix`/etc/autojump.sh
			  fi
			fi
			
			# victor @ localhost in ~/autojump on git:master o [15:08:06] 
			$ 
	

		autojump.plugin.zsh会去检查`autojump`这个命令是否存在，存在的话，才会去加载autojump。
		
		加载的动作其实就是去执行`$HOME/.autojump/etc/profile.d/autojump.zsh`这个文件。这个和我们上面执行`./install.py`时,提示我们在.zshrc中加几行代码是一样的。所以我上面说没必要再在.zshrc中加那几行代码。

	- 为所有用户安装了autojump，也就是`./install.py -s`


		这样会导致.zshrc文件中的plugins那行代码无法控制autojump的enable或者disable。安装脚本默认在`/etc/profile.d/`目录中添加了`autojump.zsh`文件。这样只要你登录账户，autojump就加载了。你无法通过.zshrc控制autojump的enable和disable。