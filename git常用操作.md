# git常用操作 #

## 创建分支 ##

	victordong@victordong-pc0 MINGW64 /d/code/liaoning_tdsql_shark (dev-victor)
	$ git branch victortest
	
	victordong@victordong-pc0 MINGW64 /d/code/liaoning_tdsql_shark (dev-victor)
	$ git branch
	* dev-victor
	  develop
	  master
	  victortest

## 展示所有分支 ##


	victordong@victordong-pc0 MINGW64 /d/code/liaoning_tdsql_shark (victortest)
	$ git branch -a
	  dev-victor
	  develop
	  master
	* victortest
	  remotes/origin/HEAD -> origin/master
	  remotes/origin/dev-victor
	  remotes/origin/develop
	  remotes/origin/develop-jefferyyin
	  remotes/origin/develop_andelwu
	  remotes/origin/develop_chasonyou_v1
	  remotes/origin/develop_goslingwang
	  remotes/origin/develop_jadenchi
	  remotes/origin/master
	  remotes/origin/victorrrrrr
	
	victordong@victordong-pc0 MINGW64 /d/code/liaoning_tdsql_shark (victortest)
	$
## 切换分支 ##

	victordong@victordong-pc0 MINGW64 /d/code/liaoning_tdsql_shark (victortest)
	$ git checkout dev-victor
	Switched to branch 'dev-victor'
	Your branch is up-to-date with 'origin/dev-victor'.
	
	victordong@victordong-pc0 MINGW64 /d/code/liaoning_tdsql_shark (dev-victor)
	$

## 展示当前分支 ##

	victordong@victordong-pc0 MINGW64 /d/code/liaoning_tdsql_shark (dev-victor)
	$ git branch
	* dev-victor
	  develop
	  master
	  victortest
	
	victordong@victordong-pc0 MINGW64 /d/code/liaoning_tdsql_shark (dev-victor)
	$
## 展示远端仓库地址 ##

	victordong@victordong-pc0 MINGW64 /d/code/liaoning_tdsql_shark (dev-victor)
	$ git remote -v
	origin  http://git.code.oa.com/QCloudCDB/tdsql_shark.git (fetch)
	origin  http://git.code.oa.com/QCloudCDB/tdsql_shark.git (push)
	
	victordong@victordong-pc0 MINGW64 /d/code/liaoning_tdsql_shark (dev-victor)
	$
	
## 添加远程仓库 ##

## 推送到远端 ##

可以将本地分支A推送到远端仓库。

如果远端仓库没有分支A，那么就在远端创建分支A；
如果远端也有分支A，那么就将本地分支A的变更推到远端仓库。

	victordong@victordong-pc0 MINGW64 /d/code/testgit (new)
	$ git push vicorigin new
	Username for 'http://git.code.oa.com': victordong
	Everything up-to-date
	
	victordong@victordong-pc0 MINGW64 /d/code/testgit (new)
	$


上面示例中，vicorigin表示远端仓库的别名，new表示本地的分支

	victordong@victordong-pc0 MINGW64 /d/code/testgit (new)
	$ git remote -v
	vicorigin       http://git.code.oa.com/QCloudCDB/tdsql_shark.git (fetch)
	vicorigin       http://git.code.oa.com/QCloudCDB/tdsql_shark.git (push)
	
	victordong@victordong-pc0 MINGW64 /d/code/testgit (new)
	$ git branch -a
	  dev-victor
	  master
	* new
	  remotes/vicorigin/dev-victor
	  remotes/vicorigin/develop
	  remotes/vicorigin/develop-jefferyyin
	  remotes/vicorigin/develop_andelwu
	  remotes/vicorigin/develop_chasonyou_v1
	  remotes/vicorigin/develop_goslingwang
	  remotes/vicorigin/develop_jadenchi
	  remotes/vicorigin/master
	  remotes/vicorigin/new
	  remotes/vicorigin/victorrrrrr
	
	victordong@victordong-pc0 MINGW64 /d/code/testgit (new)
	$

## 将一个分支和远端一个分支建立关联 ##

隆重介绍这个命令：

**git branch --set-upstream-to**

比如远程库A上有3个分支branch1、branch2、branch3。

远程库B上有3个分支branchx、branchy、branchz。

本地仓库有2个分支local1和local2。

当初始状态时，local1和local2和任何一个分支都没有关联，也就是没有upstream。

通过`git branch --set-upstream-to A/branch1 local1`命令执行后，会给local1和branch1两个分支建立关联，也就是说local1的upstream指向的是branch1。

这样的好处就是在local1分支上执行git push（git pull同理）操作时不用附加其它参数，Git就会自动将local1分支上的内容push到branch1上去。

upstream与有几个远程库没有关系，它是分支与分支之间的流通道。

参考连接：[这里](https://www.zhihu.com/question/20019419)

还有一个命令和`git branch --set-upstream-to`类似，那就是`git push -u`。

举个例子：我要把本地分支mybranch1与远程仓库origin里的分支mybranch1建立关联。

两个途径：

1. git push -u origin mybranch1
2. git branch --set-upstream-to=origin/mybranch1 mybranch1