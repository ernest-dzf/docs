# linux目录权限问题 #

- r (read contents in directory)：

	表示具有读取目录结构列表的权限，所以当你具有读取(r)一个目录的权限时，表示你可以查询该目录下的文件名数据。 所以你就可以利用 ls 这个指令将该目录的内容列表显示出来！
	
	但是你不一定能够cd到这个目录下面去，这需要(x)权限。
	
		# victor @ localhost in ~ [1:24:33] 
		$ ls -l test 
		ls: 无法访问test/a.txt: 权限不够
		总用量 0
		-????????? ? ? ? ?            ? a.txt
		
		# victor @ localhost in ~ [1:24:44] C:1
		$ ls -l
		总用量 0
		dr--r--r--. 2 victor victor 19 12月  1 23:39 test
		
		# victor @ localhost in ~ [1:24:48] 
		$ 

	victor对此目录仅具有r的权限，因此victor可以查询此目录下的文件名列表。

		# victor @ localhost in ~ [1:24:48] 
		$ chmod a+wx test 
		
		# victor @ localhost in ~ [1:27:16] 
		$ ls -l test/a.txt 
		-rwxrwxrwx. 1 victor victor 6 12月  2 00:04 test/a.txt
		
		# victor @ localhost in ~ [1:27:21] 
		$ chmod a-wx test 
		
		# victor @ localhost in ~ [1:27:32] 
		$ ls -l test/a.txt
		ls: 无法访问test/a.txt: 权限不够
		
		# victor @ localhost in ~ [1:28:55] C:1
		$ cat test/a.txt 
		cat: test/a.txt: 权限不够
		
		# victor @ localhost in ~ [1:28:59] C:1
		$ 


	从上面可以看到，victor对于test目录下的`a.txt`文件是有读写权限的，但是对于目录test只有读权限的话，是无法使用命令操作该文件的。

	这和目录的执行权限（x）有关。

- x (access directory)：

	还是上面的例子，我们给test目录添加（x）权限。

		# victor @ localhost in ~ [1:30:13] 
		$ chmod a+x test 
		
		# victor @ localhost in ~ [1:30:18] 
		$ ls -l test/a.txt 
		-rwxrwxrwx. 1 victor victor 6 12月  2 00:04 test/a.txt
		
		# victor @ localhost in ~ [1:30:26] 
		$ cat test/a.txt 
		hello
		
		# victor @ localhost in ~ [1:31:44] 
		$ 
	
	可以看到，我们可以操作a.txt文件了。

	目录的x代表的是用户能否进入该目录。

	使用者在某个目录内读取一个文件，需要使用者有目录的执行权限，文件的读权限。

- w (modify contents of directory)：

	这个可写入的权限对目录来说，是很了不起的！ 因为他表示你具有改动该目录结构列表的权限，也就是底下这些权限：

	1. 建立新的文件与目录；
	2. 删除已经存在的文件与目录(不论该文件的权限为何！)
	3. 将已存在的文件或目录进行更名；
	4. 搬移该目录内的文件、目录位置。

	针对第二点，举个例子。

	假设有个账号名称为victor，他的家目录在/home/victor/，victor对此目录具有[rwx]的权限。 若在此目录下有个名为the_root.data的文件，该文件的权限如下：

		rw-rw----. 1 root root 0 12月  2 01:37 the_root.data

	请问victor对此文件的权限为何？可否删除此文件？

	答：
	如上所示，由于victor对此文件来说是『others』的身份，因此这个文件他无法读、无法编辑也无法执行， 也就是说，他无法变动这个文件的内容就是了。
 
	但是由于这个文件在他的家目录下， 他在此目录下具有rwx的完整权限，因此对于the_root.data这个『档名』来说，他是能够『删除』的！ 结论就是，victor这个用户能够删除the_root.data这个文件！

		# victor @ localhost in ~ [1:38:24] 
		$ ls
		test  the_root.data
		
		# victor @ localhost in ~ [1:38:25] 
		$ ls -l the_root.data 
		-rw-rw----. 1 root root 0 12月  2 01:37 the_root.data
		
		# victor @ localhost in ~ [1:38:28] 
		$ cat the_root.data 
		cat: the_root.data: 权限不够
		
		# victor @ localhost in ~ [1:38:31] C:1
		$ ls -ld /home/victor
		drwx------. 6 victor victor 265 12月  2 01:38 /home/victor
		
		# victor @ localhost in ~ [1:38:56] 
		$ rm the_root.data 
		rm：是否删除有写保护的普通空文件 "the_root.data"？yes
		
		# victor @ localhost in ~ [1:39:08] 
		$ ls -l
		总用量 0
		dr-xr-xr-x. 2 victor victor 19 12月  1 23:39 test
		
		# victor @ localhost in ~ [1:39:11] 
		$ 

针对上述问题，总结一下：

1. 让使用者能进入某目录成为『可工作目录』的基本权限为何：

 - 可使用的命令：例如 cd 等变换工作目录的命令；
 - 目录所需权限：使用者对这个目录至少需要具有 x 的权限；
 - 额外需求：如果使用者想要在这个目录内利用 ls 查阅档名，则使用者对此目录还需要 r 的权限。

2. 使用者在某个目录内读取一个文件的基本权限为何？

 - 可使用的命令：例如本章谈到的 cat, more, less等等；
 - 目录所需权限：使用者对这个目录至少需要具有 x 权限；
 - 文件所需权限：使用者对文件至少需要具有 r 的权限才行！

3. 让使用者可以修改一个文件的基本权限为何？

 - 可使用的命令：例如 nano 或 vi 编辑器等；
 - 目录所需权限：使用者在该文件所在的目录至少要有 x 权限；
 - 文件所需权限：使用者对该文件至少要有 r, w 权限。

4. 让一个使用者可以创建一个文件的基本权限为何？

 - 目录所需权限：使用者在该目录要具有 w,x 的权限，重点在 w 啦！

5. 让使用者进入某目录并运行该目录下的某个命令之基本权限为何？

 - 目录所需权限：使用者在该目录至少要有 x 的权限；
 - 文件所需权限：使用者在该文件至少需要有 x 的权限。