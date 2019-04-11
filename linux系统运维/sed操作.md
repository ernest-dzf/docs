# sed常用操作 #

sed的语法：

**sed [options] 'command' filename(s)**

sed的寻址方式：

1. 单行寻址：［line-address］command
	
	寻找匹配line-address的行并进行处理，比如，`sed -n '2p' sedB.txt `，打印sedB.txt文件的第二行。

2. 行集合寻址：［regexp］command 
	匹配文件中的一行或多行，比如，`sed '/^m/ s/is/are/g' sedB.txt`，将sedB.txt文件中的以`m`开头的行中的`is`替换位`are`。

3. 多行寻址： [line-address1，line-address2] command

	寻找在两个地址之间的内容并做相应的处理。


先准备几个测试文件。

测试文件A:

	# victordong @ VM_144_188_centos in /data/victor [14:08:20] $ cat sedA.txt 
	my cat's name is betty
	this is your dog
	my dog's name is frank
	this is your fish
	my fish's name is george
	this is your goat
	my goat's name is adam
	
	# victordong @ VM_144_188_centos in /data/victor [14:15:21] 
	$ 

测试文件B:

	# victordong @ VM_144_188_centos in /data/victor [14:18:20] 
	$ cat sedB.txt 
	my cat's name is betty
	this is your dog. this is you cat.
	my dog's name is frank
	this is your fish
	my fish's name is george
	this is your goat
	my goat's name is adam
	
	# victordong @ VM_144_188_centos in /data/victor [14:18:25] 
	$ 


## 替换 ##

1. 将sedB.txt文件中的**所有**`this`替换位`This`.

		# victordong @ VM_144_188_centos in /data/victor [14:18:25] 
		$ sed 's/this/This/g' sedB.txt
		my cat's name is betty
		This is your dog. This is you cat.
		my dog's name is frank
		This is your fish
		my fish's name is george
		This is your goat
		my goat's name is adam
		
		# victordong @ VM_144_188_centos in /data/victor [14:18:50] 
		$ 

2. 将sedB.txt文件中的每一行的**第一个**`this`替换为`This`.

		# victordong @ VM_144_188_centos in /data/victor [14:20:03] C:1
		$ sed 's/this/This/' sedB.txt
		my cat's name is betty
		This is your dog. this is you cat.
		my dog's name is frank
		This is your fish
		my fish's name is george
		This is your goat
		my goat's name is adam
		
		# victordong @ VM_144_188_centos in /data/victor [14:20:09] 
		$ 

3. 将sedB.txt文件中的以`m`开头的行中的`is`替换位`are`.

		# victordong @ VM_144_188_centos in /data/victor [15:21:48] 
		$ sed '/^m/ s/is/are/g' sedB.txt
		my cat's name are betty
		this is your dog. this is you cat.
		my dog's name are frank
		this is your fish
		my fareh's name are george
		this is your goat
		my goat's name are adam
		
		# victordong @ VM_144_188_centos in /data/victor [15:25:28] 
		$ 

4. 将sedB.txt文件中的第1到第3行中的`is`替换位`are`.

		# victordong @ VM_144_188_centos in /data/victor [15:28:02] C:1
		$ sed '1,3 s/is/are/g' sedB.txt
		my cat's name are betty
		thare are your dog. thare are you cat.
		my dog's name are frank
		this is your fish
		my fish's name is george
		this is your goat
		my goat's name is adam
		
		# victordong @ VM_144_188_centos in /data/victor [15:29:06] 
		$ 

## 删除 ##

1. 删除`sedB.txt`中以`my`开头的行
	
		# victordong @ VM_144_188_centos in /data/victor [20:20:37] 
		$ sed '/my /d' sedB.txt
		this is your dog. this is you cat.
		this is your fish
		this is your goat
