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

通过`git branch --set-upstream-to=A/branch1 local1`命令执行后，会给local1和branch1两个分支建立关联，也就是说local1的upstream指向的是branch1。

这样的好处就是在local1分支上执行git push（git pull同理）操作时不用附加其它参数，Git就会自动将local1分支上的内容push到branch1上去。

upstream与有几个远程库没有关系，它是分支与分支之间的流通道。

参考连接：[这里](https://www.zhihu.com/question/20019419)

还有一个命令和`git branch --set-upstream-to`类似，那就是`git push -u`。

举个例子：我要把本地分支mybranch1与远程仓库origin里的分支mybranch1建立关联。

两个途径：

1. git push -u origin mybranch1
2. git branch --set-upstream-to=origin/mybranch1 mybranch1

这两种方式都可以达到目的。但是方法1更通用，因为你的远程库有可能并没有mybranch1分支，这种情况下你用方法2就不可行，连目标分支都不存在，怎么进行关联呢？

所以可以总结一下：`git push -u origin mybranch1` 相当于  `git push origin mybranch1` + `git branch --set-upstream-to=origin/mybranch1 mybranch1`

## git add ##

放到暂存区

	# victor @ VICTORDONG-MB0 in ~/github/notes on git:master x [2:06:09] 
	$ # victor @ VICTORDONG-MB0 in ~/github/notes on git:master x [2:05:31] 
	$ git status
	On branch master
	Your branch is up to date with 'origin/master'.
	
	Changes not staged for commit:
	  (use "git add <file>..." to update what will be committed)
	  (use "git checkout -- <file>..." to discard changes in working directory)
	
	        modified:   git常用操作.md
	
	no changes added to commit (use "git add" and/or "git commit -a")
	
	# victor @ VICTORDONG-MB0 in ~/github/notes on git:master x [2:05:34]
	$ git add git常用操作.md
	
## git commit ##
提交

	# victor @ VICTORDONG-MB0 in ~/github/notes on git:master x [2:08:18] 
	$ git commit -m 'description'
	
## git checkout ##

modified的文件，使用`git checkout -- <file>`命令可以将更改丢弃。

## 解决冲突 ##

git 冲突文件是类似 xxx.orig 这种的文件，以orig为后缀。

	<<<<<<< HEAD
	b789
	=======
	b45678910
	>>>>>>> 6853e5ff961e684d3a6c02d4d06183b5ff330dcc

上面的例子中，`<<<<<<< HEAD`到`=======`之间的`b789`是本地的文件，`=======`到`>>>>>>> 6853e5ff961e684d3a6c02d4d06183b5ff330dcc`之间的`b45678910`是拉下来的文件。

手动解决一下冲突，然后保存就可以。

xxx.orig文件是冲突现场，解决完冲突后，可以选择将其删除掉。

## git show ##

查看某次commit做了哪些更改，使用`git show xxxx`命令，其中`xxxx`表示commit id。

	$ git show b75bf88a841c070526203a62c415b56d1e30b61f
	commit b75bf88a841c070526203a62c415b56d1e30b61f (HEAD -> tdsql-refactor-develop, origin/tdsql-refactor-develop)
	Author: victordong <victordong@tencent.com>
	Date:   Mon Apr 15 18:05:07 2019 +0800
	
	    fix uniqSubnetId&uniqVpcId parameter
	
	diff --git a/src/comcomponent/vpc.go b/src/comcomponent/vpc.go
	index 167f544..5ee5a63 100644
	--- a/src/comcomponent/vpc.go
	+++ b/src/comcomponent/vpc.go
	@@ -81,8 +81,8 @@ func NewVpc(region string, requestId string) *Vpc {
	 }
	
	 type VpcApplyIpRequest struct {
	-       UniqVpcId       string  `json:"uniqvpcId,omitempty"`
	-       UniqSubnetId    string  `json:"uniqVpcId,omitempty"`
	+       UniqVpcId       string  `json:"uniqVpcId,omitempty"`
	+       UniqSubnetId    string  `json:"uniqSubnetId,omitempty"`
	
	        VpcId    int64  `json:"vpcId"`
	        SubnetId int64  `json:"subnetId"`
	@@ -381,8 +381,8 @@ type GetSubnetIpNumResponse struct {
	        Detail []SubnetIpNum `json:"detail"`
	 }
	 type SubnetIpNum struct {
	-       UniqVpcId    string `json:"uniqvpcId,omitempty"`
	-       UniqSubnetId string `json:"uniqVpcId,omitempty"`
	+       UniqVpcId    string `json:"uniqVpcId,omitempty"`
	+       UniqSubnetId string `json:"uniqSubnetId,omitempty"`
	        VpcId        int    `json:"vpcId"`
	        SubnetId     int    `json:"subnetId"`
	        Used         int    `json:"used"`
	
	victordong@victordong-pc0 MINGW64 /d/code/liaoning_tdsql_shark/deps/common (tdsql-refactor-develop)
	$
