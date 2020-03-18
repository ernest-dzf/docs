# zsh安装
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

# 改变ls命令执行后的文件颜色

1. 下载`dircolors-solarized`，github上clone这个地

   ```shell
   git clone https://github.com/seebi/dircolors-solarized.git
   ```

2. 应用某个主题

   ```shell
   # root @ localhost in ~/tools/dircolors-solarized on git:master o [9:31:28]
   $ ls
   dircolors.256dark    dircolors.ansi-light      img       README.md
   dircolors.ansi-dark  dircolors.ansi-universal  Makefile  test-directory.tar.bz2
   
   # root @ localhost in ~/tools/dircolors-solarized on git:master o [9:31:29]
   $
   ```

   上面`dircolors-solarized`目录就是下载得到的主题文件，里面有多个主题。我们应用其中某个主题，比如`dircolors.256dark`。

   ```shell
   # root @ localhost in ~/tools/dircolors-solarized on git:master o [9:36:20]
   $ ls
   dircolors.256dark    dircolors.ansi-light      img       README.md
   dircolors.ansi-dark  dircolors.ansi-universal  Makefile  test-directory.tar.bz2
   
   # root @ localhost in ~/tools/dircolors-solarized on git:master o [9:36:22]
   $ cp dircolors.256dark ~/.dircolors
   
   # root @ localhost in ~/tools/dircolors-solarized on git:master o [9:36:49]
   $ ls -l ~/.dircolors
   -rw-r--r--. 1 root root 7300 3月   3 09:36 /root/.dircolors
   
   # root @ localhost in ~/tools/dircolors-solarized on git:master o [9:36:54]
   $
   ```

   同时在你的`.zshrc`中，添加如下，

   ```shell
   eval `dircolors /root/.dircolors`
   ```

3. 重新登录既可

#oh-my-zsh安装


oh-my-zsh 帮我们整理了一些常用的 zsh 扩展功能和主题。

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

```shell
usermod -s /bin/zsh root
```
然后，将victor主目录下面的`.oh-my-zsh`和`.zshrc`拷贝到root主目录下面去。注意修改`.zshrc`中的	ZSH变量。

```shell
……
# Path to your oh-my-zsh installation.
  export ZSH="/root/.oh-my-zsh"

# Set name of the theme to load --- if set to "random", it will
……
```

修改为root用户的.oh-my-zsh所在的目录。

然后重新登录root用户，就可以了。

# zsh一些插件 #

## autojump ##

autojump是一个命令行工具，它允许你可以直接跳转到你喜爱的目录，而不用管你现在身在何处。

1. 安装autojump
	
	先下载：
	```shell
	https://github.com/wting/autojump.git
	```
	安装：
	```shell
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
	```
	执行`autojump`目录下的install.py文件即可，
	```	shell
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
	
	# victor @ localhost in ~/autojump on git:master o [14:59:12] 
	$ 
	```
	
	安装过程有提示说，让在.zshrc文件中添加几行，其实没必要。
	
	如果你没有使用oh-my-zsh，你可以按照上面的提示去做。
	
	安装了oh-my-zsh的话，我们可以利用oh-my-zsh的插件管理来管理我们的autojump。只需要在plugins中设置加载autojump即可。

2. 编辑.zshrc文件，找到类似下面的几行
	
	```shell
	……
	# Example format: plugins=(rails git textmate ruby lighthouse)
	# Add wisely, as too many plugins slow down shell startup.
	plugins=(
	git autojump
	)
	autoload -U compinit && compinit -u
	source $ZSH/oh-my-zsh.sh
	
	# User configuration
	……
	```
	在`plugins=(git)`后面添加autojump，即表示加载autojump插件，同时添加`autoload -U compinit && compinit -u`。
	
	保存退出，重新登录账户，或者source一下.zshrc文件即可。

3. 踩到的坑

	- 没有安装autojump，就直接在.zshrc文件中plugins中设置加载autojump，这肯定不得行
	
4. 背后的逻辑

   ```shell
   # root @ localhost in ~/.oh-my-zsh/plugins/autojump on git:master o [16:07:42]
   $ ls -l autojump.plugin.zsh
   -rw-r--r--. 1 root root 1212 3月   3 10:12 autojump.plugin.zsh
   
   # root @ localhost in ~/.oh-my-zsh/plugins/autojump on git:master o [16:07:50]
   $
   
   ```

   `.zshrc`文件中 plugins配置启用了autojump的话，会去`~/.oh-my-zsh/plugins/autojump`目录执行`autojump.plugin.zsh`文件，加载autojump。

   加载autojump最终还是落脚到去执行`.autojump/etc/profile.d/autojump.sh`这个文件。

   ```shell
   declare -a autojump_paths
   autojump_paths=(
     $HOME/.autojump/etc/profile.d/autojump.zsh         # manual installation
     $HOME/.autojump/share/autojump/autojump.zsh        # manual installation
     $HOME/.nix-profile/etc/profile.d/autojump.sh       # NixOS installation
     /run/current-system/sw/share/autojump/autojump.zsh # NixOS installation
     /usr/share/autojump/autojump.zsh                   # Debian and Ubuntu package
     /etc/profile.d/autojump.zsh                        # manual installation
     /etc/profile.d/autojump.sh                         # Gentoo installation
     /usr/local/share/autojump/autojump.zsh             # FreeBSD installation
     /opt/local/etc/profile.d/autojump.sh               # macOS with MacPorts
     /usr/local/etc/profile.d/autojump.sh               # macOS with Homebrew (default)
   )
   
   for file in $autojump_paths; do
     if [[ -f "$file" ]]; then
       source "$file"
       found=1
       break
     fi
   done
   
   # if no path found, try Homebrew
   if (( ! found && $+commands[brew] )); then
     file=$(brew --prefix)/etc/profile.d/autojump.sh
     if [[ -f "$file" ]]; then
       source "$file"
       found=1
     fi
   fi
   
   (( ! found )) && echo '[oh-my-zsh] autojump script not found'
   
   unset autojump_paths file found
   ```

   