# tmux #

1. ncurses-devel 需要安装。

		yum -y install ncurses-devel

2. libevent 安装
		
	
		wget https://github.com/libevent/libevent/releases/download/release-2.0.22-stable/libevent-2.0.22-stable.tar.gz
		
		tar -xzvf libevent-2.0.22-stable.tar.gz
		
		cd libevent-2.0.22-stable
		
		./configure
		
		make && make install

	安装完成后，查看一下libevent的安装路径，一般下载源码安装的会安装在`/usr/local/lib`目录下面，需要将这个目录加入到共享库配置文件`/etc/ld.so.conf`中，然后执行ldconfig命令。
		
		echo "/usr/local/lib" >> /etc/ld.so.conf
		ldconfg 

3. tmux安装

	从github上下载最新源码安装。

		git clone https://github.com/tmux/tmux.git
		cd tmux
		sh autogen.sh
		./configure && make
	
	使用源码安装的话，需要一些依赖项目。比如gcc，make，autoconf，automake，pkg-config等。

4. tmu的基本概念
	- session，会话。会话可以通过 `tmux new -s session1` 创建，其中 session1表示会话的名字。每次在linux命令行敲击 `tmux` 进入tmux时，片会创建一个会话。
	- window，窗口。一个会话可以有多个窗口。
	- pane，窗格。一个窗口可以有多个窗格，也是我们操作的最小单位。我们一般都是在窗格上进行工作。

	下图展示的即是tmux中使用`ctrl + b, w`的效果，展示了session、windows和pane的树形关系。

	![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/tmux_session.png)

	下图展示的是tmux的布局说明图。

	![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/tmux_session_window_pane.png)

5. tmux常用操作

	tmux中使用的命令都是以`ctrl + b`开头。	

	- tmux new -s session1 新建会话
	- ctrl+b d 退出会话，回到shell的终端环境
	- tmux ls 终端环境查看会话列表
	- ctrl+b s 会话环境查看会话列表
	- tmux attach 从终端进入会话
	- tmux a -t session1 从终端环境进入指定会话
	- tmux kill-session -t session1 销毁会话
	- tmux rename -t old_session_name  new_session_name  重命名会话
	- ctrl + b $ 重命名会话 (在会话环境中)
	- ctrl + b c 创建window
	- ctrl + b p 切换到上一个window
	- ctrl + b n 切换到下一个window
	- ctrl + b 0 切换到0号window，依次类推，可换成任意窗口序号
	- ctrl + b w (windows的首字母) 列出当前session所有window，通过上、下键切换窗口
	- ctrl + b l (字母L的小写)相邻的window切换
	- ctrl + b & 关闭当前window，会给出提示是否关闭当前窗口，按下y确认即可
	- ctrl + b % 垂直分屏(组合键之后按一个百分号)，用一条垂线把当前窗口分成左右两屏
	- ctrl + b " 水平分屏(组合键之后按一个双引号)，用一条水平线把当前窗口分成上下两屏
	- ctrl + b o 依次切换当前窗口下的各个pane
	- ctrl + b Up|Down|Left|Right 根据按箭方向选择切换到某个pane
	- ctrl + b z 最大化当前pane，再按一次后恢复
	- ctrl + b x 关闭当前使用中的pane，操作之后会给出是否关闭的提示，按y确认即关闭
	- ctrl + b [ 进入copy mode，然后可以使用pageUp和pageDown键scroll了，或者使用上下左右方向键，退出copy mode 按`q`就可以
	- tmux kill-session -t session 退出某一个session，这会关闭这个session所有的window，并退出 session

6. 配置

	tmux的配置文件为`.tmux.conf`，可以放到用户的主目录。

		# victordong @ VM_144_188_centos in ~ [23:47:33] 
		$ ls -l .tmux.conf 
		-rw-r--r-- 1 victordong users 60 Oct 27 17:34 .tmux.conf
		
		# victordong @ VM_144_188_centos in ~ [23:47:41] 
		$ 


- **固定window的名字** 
	
		有时候，想固定一个window的名字，可以这样
	
	```shell
		# victordong @ VM_144_188_centos in ~ [23:47:41] 
		$ cat .tmux.conf 
		setw -g automatic-rename off
		set-option -g allow-rename off
		
		# victordong @ VM_144_188_centos in ~ [23:52:36] 
		$ 
	```



7. 移动pane

   可以使用`ctrl b {`或者`ctrl b }`在不同的pane之间移动。

   

8. 在不同的pane之间批量操作。
   `ctrl b`，然后执行`: set synchronize-panes on`开启批量操作。

8. 参考文章

  [linux安装tmux](https://blog.csdn.net/lijing742180/article/details/80663878 "linux安装tmux")

  [tmux终端复用详解](https://www.cnblogs.com/wangqiguo/p/8905081.html#_labelTop "tmux终端复用详解")

