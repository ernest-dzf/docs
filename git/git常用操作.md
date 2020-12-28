# git常用操作 

## git代理

- 查看当前代理

  ```
  git config --global http.proxy
  ```

  

- 设置当前代理

  ```ht
  git config --global http.proxy 'http://127.0.0.1:1080'
  git config --global https.proxy 'https://127.0.0.1:1080'
  ```

- 删除代理

  ```
  git config --global --unset http.proxy
  git config --global --unset https.proxy
  ```

- 针对当前项目设置代理

  ```
  git config --local https.proxy https://127.0.0.1:12639
  ```

- 针对某个域名设置代理

  ```shell
  git config --global http.https://github.com.proxy http://127.0.0.1:12639
  ```

  


## git配置文件

git里面一共有3个配置文件。

首先是：仓库级配置文件。该文件位于当前仓库下，路径.git/，文件名为.gitconfig，这个配置中的设置只对当前所在仓库有效。

然后是：全局配置文件。作用于所有用户 ，级别高于系统级配置文件。

最后是：系统级别配置文件。作用于系统所有用户和所有库。

三个级别的配置文件分别可以通过下面这三个命令进行编辑。

```
git config -e
git config -e --global
git config -e --system
```



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
## git 更新远程分支列表 ##

远端创建了分支，但是使用`git branch -a`看不到远端分支信息，导致不能checkout该分支。

	 git remote update origin --prune

## git 恢复到之前某一次提交 ##

有时候本地commit，但是不想把这个commit提交到远端，可以使用`git reset --hard xxxx`，然后再进行更改，push。

## git 查看stage的变更

```
git diff --staged
```

## git查看2次commit的不同

```
git diff commit-1 commit-2
```

commit-1表示前一次提交，commit-2表示后一次提交。

上面命令将展示出commit-2相对于commit-1的不同之处。

## git查看当前commit 版本号

```
git rev-parse HEAD
```

## git merge合并代码
- 开发分支（dev）上的代码达到上线的标准后，要合并到 master 分支

  ```shell
  git checkout dev
  git pull
  git checkout master
  git merge dev
  git push -u origin master
  ```

- 当master代码改动了，需要更新开发分支（dev）上的代码

	```shell
	git checkout master 
  git pull 
  git checkout dev
  git merge master 
  git push -u origin dev
	```

## git stash

当前分支工作区间已经修改了一些类容，但是你不想提交。这时候你直接`git checkout xxxx`是不能切到另外一个分支的。使用`git stash`可以解决。

```
git stash [save message]
```

这样工作目录就干净了，切到另外一个分支之后，你可以使用

```
git stash pop
```

将stash栈顶的内容应用到当前工作目录上，**只能恢复一次**，回复之后对应的贮藏就会被从栈顶拿走。

你可以认为stash有一个栈，使用

```
git stash list
```

可以查看，

```
stash@{0}: On master: configuration file
(END)
```

你也可以删除某个贮藏，

```
git stash drop stash@{num}
```

你也可以将整个stash清除，

```
git stash clear
```

你也可以单独应用某个stash，

```
git stash apply stash@{num}
```

num为你使用`git stash list`看到的数字。

## 统计本地分支和跟踪的远程分支的差异commit

先更新本地的远程分支，`git fetch origin`，

然后比较本地分支和远程分支的差异，

`git log master..origin/master`

## 如何知道当前分支是从哪个分支的哪个commit长出来的

```
git reflog show branch-name
```

## git checkout -b

`git checkout -b mybranch`，创建并切换到分支mybranch。

记住，这里创建并切换到分支`mybranch`是以你当前local的记录为准的。有可能你当前local记录落后origin。

如果是要以远程的分支为基准去创建，该这样做

`git checkout -b mybranch origin/mybranch`

## 配置local项目的user

有时候不想配置全局的user name和email，那么可以仅配置当前项目的user和email。

```
git config user.name wind
git config user.email xxxx@qq.com
```

