# linux 硬链接 软链接 #

我们知道文件都有文件名与数据，这在 Linux 上被分成两个部分：用户数据 (user data) 与元数据 (metadata)。

用户数据，即文件数据块 (data block)，数据块是记录文件真实内容的地方；而元数据则是文件的附加属性，如文件大小、创建时间、所有者等信息。

在 Linux 中，元数据中的 inode 号（inode 是文件元数据的一部分但其并不包含文件名，inode 号即索引节点号）才是文件的唯一标识而非文件名。文件名仅是为了方便人们的记忆和使用，系统或程序通过 inode 号寻找正确的文件数据块。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/link2.jpg)

文件的元数据可以使用`stat`命令查看，比如：

	# root @ localhost in ~ [14:52:29] 
	$ stat test.txt 
	  文件："test.txt"
	  大小：0         	块：0          IO 块：4096   普通空文件
	设备：fd00h/64768d	Inode：71497137    硬链接：1
	权限：(0664/-rw-rw-r--)  Uid：(    0/    root)   Gid：(    0/    root)
	环境：unconfined_u:object_r:admin_home_t:s0
	最近访问：2018-12-01 14:52:29.429419478 +0800
	最近更改：2018-12-01 14:52:29.429419478 +0800
	最近改动：2018-12-01 14:52:29.429419478 +0800
	创建时间：-
	
	# root @ localhost in ~ [14:52:32] 

linux中使用`mv`移动并重命名文件，并不影响文件的用户数据及inode号，比如：

	# root @ localhost in ~ [14:54:38] 
	$ stat test.txt 
	  文件："test.txt"
	  大小：0         	块：0          IO 块：4096   普通空文件
	设备：fd00h/64768d	Inode：71497137    硬链接：1
	权限：(0664/-rw-rw-r--)  Uid：(    0/    root)   Gid：(    0/    root)
	环境：unconfined_u:object_r:admin_home_t:s0
	最近访问：2018-12-01 14:52:29.429419478 +0800
	最近更改：2018-12-01 14:52:29.429419478 +0800
	最近改动：2018-12-01 14:52:29.429419478 +0800
	创建时间：-
	
	# root @ localhost in ~ [14:54:47] 
	$ mv test.txt test2.txt
	
	# root @ localhost in ~ [14:55:07] 
	$ stat test2.txt 
	  文件："test2.txt"
	  大小：0         	块：0          IO 块：4096   普通空文件
	设备：fd00h/64768d	Inode：71497137    硬链接：1
	权限：(0664/-rw-rw-r--)  Uid：(    0/    root)   Gid：(    0/    root)
	环境：unconfined_u:object_r:admin_home_t:s0
	最近访问：2018-12-01 14:52:29.429419478 +0800
	最近更改：2018-12-01 14:52:29.429419478 +0800
	最近改动：2018-12-01 14:55:07.516800498 +0800
	创建时间：-
	
	# root @ localhost in ~ [14:55:12] 
	$ 

从上面可以看到，test.txt和test2.txt文件的inode号，均为71497137。

为解决文件的共享使用，Linux 系统引入了两种链接：硬链接 (hard link) 与软链接（又称符号链接，即 soft link 或 symbolic link）。

链接为 Linux 系统解决了文件的共享使用，还带来了隐藏文件路径、增加权限安全及节省存储等好处。

## 硬链接 ##

若一个 inode 号对应多个文件名，则称这些文件为硬链接。换言之，硬链接就是同一个文件使用了多个别名。

硬链接可由命令 link 或 ln 创建。如下是对文件 oldfile 创建硬链接。

	# root @ localhost in ~ [14:55:12] 
	$ touch oldfile
	
	# root @ localhost in ~ [14:59:46] 
	$ ln oldfile newfile
	
	# root @ localhost in ~ [14:59:51] 
	$ ls -l
	总用量 681848
	-rw-------.  1 root root      1423 12月  1 08:31 anaconda-ks.cfg
	-rwxrwxr-x.  1 root root 312845162 12月  1 13:22 mysql-5.6.27-linux-glibc2.5-x86_64.tar.gz
	drwxrwxr-x. 13 root root       191 12月  1 13:03 mysql-5.6.42-linux-glibc2.12-x86_64
	-rwxrwxr-x.  1 root root 328979165 12月  1 13:03 mysql-5.6.42-linux-glibc2.12-x86_64.tar.gz
	-rw-rw-r--.  2 root root         0 12月  1 14:59 newfile
	-rw-rw-r--.  2 root root         0 12月  1 14:59 oldfile
	-rw-rw-r--.  1 root root         0 12月  1 14:52 test2.txt
	-r--r--r--.  1 root root  56375699 12月  1 12:57 VMwareTools-10.1.15-6627299.tar.gz
	drwxr-xr-x.  9 root root       145 9月  14 2017 vmware-tools-distrib
	
	# root @ localhost in ~ [14:59:52] 
	$ 

对文件oldfile的修改，newfile能够同步看到，

	# root @ localhost in ~ [15:00:35] 
	$ echo "hello" > oldfile 
	
	# root @ localhost in ~ [15:00:47] 
	$ cat newfile 
	hello
	
	# root @ localhost in ~ [15:00:49] 
	$ cat oldfile 
	hello
	
	# root @ localhost in ~ [15:00:52] 
	$ 

通过`stat`命令也可以看到oldfile和newfile的inode是一样的

	# root @ localhost in ~ [15:00:52] 
	$ stat oldfile 
	  文件："oldfile"
	  大小：6         	块：8          IO 块：4096   普通文件
	设备：fd00h/64768d	Inode：71497138    硬链接：2
	权限：(0664/-rw-rw-r--)  Uid：(    0/    root)   Gid：(    0/    root)
	环境：unconfined_u:object_r:admin_home_t:s0
	最近访问：2018-12-01 15:00:49.197027523 +0800
	最近更改：2018-12-01 15:00:47.409009519 +0800
	最近改动：2018-12-01 15:00:47.409009519 +0800
	创建时间：-
	
	# root @ localhost in ~ [15:01:25] 
	$ stat newfile  
	  文件："newfile"
	  大小：6         	块：8          IO 块：4096   普通文件
	设备：fd00h/64768d	Inode：71497138    硬链接：2
	权限：(0664/-rw-rw-r--)  Uid：(    0/    root)   Gid：(    0/    root)
	环境：unconfined_u:object_r:admin_home_t:s0
	最近访问：2018-12-01 15:00:49.197027523 +0800
	最近更改：2018-12-01 15:00:47.409009519 +0800
	最近改动：2018-12-01 15:00:47.409009519 +0800
	创建时间：-
	
	# root @ localhost in ~ [15:01:29] 
	$ 

可以看到，inode都是71497138。也可以验证，oldfile和newfile其实是同一个文件，只不过文件名不一样而已。

由于硬链接是有着相同 inode 号仅文件名不同的文件，因此硬链接存在以下几点特性：

- 文件有相同的 inode 及 data block
- 只能对已存在的文件进行创建

	inode 是随着文件的存在而存在，因此只有当文件存在时才可创建硬链接，即当 inode 存在且链接计数器（link count）不为 0 时。

- 不能交叉文件系统进行硬链接的创建

	inode 号仅在各文件系统下是唯一的，当 Linux 挂载多个文件系统后将出现 inode 号重复的现象.因此硬链接创建时不可跨文件系统。

- **不能对目录进行创建，只可对文件创建**

	硬链接不能对目录创建是受限于文件系统的设计。

		# victor @ localhost in ~ [15:45:39] 
		$ ls -l
		总用量 0
		drwxrwxr-x. 2 victor victor 6 12月  1 15:44 test
		
		# victor @ localhost in ~ [15:45:40] 
		$ stat test
		  文件："test"
		  大小：6         	块：0          IO 块：4096   目录
		设备：fd00h/64768d	Inode：101817402   硬链接：2
		权限：(0775/drwxrwxr-x)  Uid：( 1000/  victor)   Gid：( 1000/  victor)
		环境：unconfined_u:object_r:user_home_t:s0
		最近访问：2018-12-01 15:44:17.926509697 +0800
		最近更改：2018-12-01 15:44:15.195454522 +0800
		最近改动：2018-12-01 15:44:15.195454522 +0800
		创建时间：-
		
		# victor @ localhost in ~ [15:45:44] 
		$ cd test 
		
		# victor @ localhost in ~/test [15:45:47] 
		$ stat .
		  文件："."
		  大小：6         	块：0          IO 块：4096   目录
		设备：fd00h/64768d	Inode：101817402   硬链接：2
		权限：(0775/drwxrwxr-x)  Uid：( 1000/  victor)   Gid：( 1000/  victor)
		环境：unconfined_u:object_r:user_home_t:s0
		最近访问：2018-12-01 15:44:17.926509697 +0800
		最近更改：2018-12-01 15:44:15.195454522 +0800
		最近改动：2018-12-01 15:44:15.195454522 +0800
		创建时间：-
		
		# victor @ localhost in ~/test [15:45:53] 
		$  


	上面例子也可以看到，目录test的inode为101817402，test下的当前目录（.）inode号也是101817402。

	Linux 文件系统中的目录均隐藏了两个特殊的目录：当前目录（.）与父目录（..）。查看这两个特殊目录的 inode 号，可知其实这两目录就是两个硬链接。这里也可以看到，**目录的硬链接技术上是可以做到的**，但是如果允许用户自己创建对目录的硬链接的话，会来带很多问题。

	考虑这样一个场景：

		mkdir -p /tmp/a/b
		cd /tmp/a/b
		ln -d /tmp/a l

	1. 打破了parent directory的无歧义性

			cd /tmp/a/b
			cd /tmp/a/b/l/b
	
		目录`/tmp/a/b`的父目录可以是`/tmp/a`，也可以是`/tmp/a/b/l`
	2. 会导致无限循环

			cd /tmp/a/b/l/b/l/b/l/b/l/b

	3. 同一个文件也会有不同的路径
			
			/tmp/a/b/foo.txt
			/tmp/a/b/l/b/foo.txt

	**其实上面的这些问题，软链接也会遇到**，但是软链接很好判别其特性，也即special, detectable, and skippable。应用程序可以比较容易地对其做特殊处理，从而避开上面的这三点。

	但是硬链接就不一样了，你创建了一个硬链接后，你是无法判断哪个是源文件，哪个是硬链接的。**因为本质上，它们就是一样的。**

	参考：[硬链接为啥不能指向目录](https://unix.stackexchange.com/questions/22394/why-are-hard-links-to-directories-not-allowed-in-unix-linux)

	

- 删除一个硬链接文件并不影响其他有相同 inode 号的文件

	若要删除一个文件，则应该将他的所有硬链接全部删除。

	换句话说，如果一个文件有硬链接，则只删除原文件或硬链接文件是无法将其删除的。

## 软链接 ##

软链接与硬链接不同，若文件用户数据块中存放的内容是另一文件的路径名的指向，则该文件就是软连接。

软链接就是一个普通文件，只是数据块内容有点特殊。软链接有着自己的 inode 号以及用户数据块。

	# root @ localhost in ~ [15:05:08] 
	$ ls -l oldfile 
	-rw-rw-r--. 2 root root 6 12月  1 15:00 oldfile
	
	# root @ localhost in ~ [15:05:15] 
	$ ln -s oldfile oldfile-softlink
	
	# root @ localhost in ~ [15:05:31] 
	$ ls -l oldfile-softlink 
	lrwxrwxrwx. 1 root root 7 12月  1 15:05 oldfile-softlink -> oldfile
	
	# root @ localhost in ~ [15:05:35] 
	$ ls -i oldfile-softlink 
	71497139 oldfile-softlink
	
	# root @ localhost in ~ [15:05:47] 
	$ ls -i oldfile
	71497138 oldfile
	
	# root @ localhost in ~ [15:05:52] 
	$ 

从上面可以看到，我们创建了一个指向oldfile的软链接oldfile-softlink。它的inode号是71497139，与oldfile的inode号（71497138）不同。

这里用示意图展示一下硬链接和软链接的不同。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/link1.jpg)

## 软硬链接的删除 ##

如果软链接或者硬链接，链接的是普通文件的话，直接删除就好了，不会删除源文件的。

	# victor @ localhost in ~/test [23:12:33] 
	$ ls -l
	总用量 0
	-rw-rw-r--. 1 victor victor 0 12月  1 22:52 a.txt
	lrwxrwxrwx. 1 victor victor 5 12月  1 23:12 b -> a.txt
	
	# victor @ localhost in ~/test [23:12:36] 
	$ rm b
	
	# victor @ localhost in ~/test [23:12:39] 
	$ ls -l
	总用量 0
	-rw-rw-r--. 1 victor victor 0 12月  1 22:52 a.txt
	
	# victor @ localhost in ~/test [23:12:40] 
	$ 

如果链接到目录的话（软链接），就需要注意点儿了。

	# victor @ localhost in ~ [23:38:32] 
	$ ls -l
	总用量 0
	lrwxrwxrwx. 1 victor victor  4 12月  1 23:14 softtest -> test
	drwxrwxr-x. 2 victor victor 19 12月  1 23:12 test
	
	# victor @ localhost in ~ [23:38:38] 
	$ ls test 
	a.txt
	
	# victor @ localhost in ~ [23:38:45] 
	$ rm -rf softtest/      
	
	# victor @ localhost in ~ [23:38:59] 
	$ ls -l test 
	总用量 0
	
	# victor @ localhost in ~ [23:39:04] 
	$ 


这里`rm -rf softtest/`之后，可以看到test目录下的文件也被删除了。

	# victor @ localhost in ~ [23:40:00] 
	$ ls -l
	总用量 0
	lrwxrwxrwx. 1 victor victor  4 12月  1 23:14 softtest -> test
	drwxrwxr-x. 2 victor victor 19 12月  1 23:39 test
	
	# victor @ localhost in ~ [23:40:04] 
	$ ls -l test 
	总用量 0
	-rw-rw-r--. 1 victor victor 0 12月  1 23:39 a.txt
	
	# victor @ localhost in ~ [23:40:09] 
	$ rm softtest 
	
	# victor @ localhost in ~ [23:40:12] 
	$ ls -l
	总用量 0
	drwxrwxr-x. 2 victor victor 19 12月  1 23:39 test
	
	# victor @ localhost in ~ [23:40:14] 
	$ ls -l test 
	总用量 0
	-rw-rw-r--. 1 victor victor 0 12月  1 23:39 a.txt
	
	# victor @ localhost in ~ [23:40:21] 
	$ 

如果使用`rm -rf softtest`，则仅仅会删除软链接。

软链接也可以使用`unlink`来删除。