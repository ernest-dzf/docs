# OverlayFS文件系统

OverlayFS是一种堆叠文件系统，它依赖并建立在其他文件系统之上（例如ext4fs和xfs等），并不直接参与磁盘空间结构的划分，仅仅将原来底层文件系统中不同的目录进行“合并”，然后向用户呈现。因此对于用户来说，它所见到的overlay文件系统根目录下的内容就来自挂载时所指定的不同目录的“合集”。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/overlayFS.jpeg)

其中Lower DirA 、 Lower DirB目录和Upper Dir目录为来自底层文件系统的不同目录，用户可以自行指定，内部包含了用户想要合并的文件和目录，Merge Dir目录为挂载点。

当文件系统挂载后，在Merge Dir目录下将会同时看到来自各Lower和Upper目录下的内容，并且用户也无法（无需）感知这些文件哪些来自Lower Dir，哪些来自Upper Dir，用户看见的只是一个普通的文件系统根目录而已（Lower Dir可以有多个也可以只有一个）。

**疑问**：Upper Dir 可以有多个么？



虽然OverlayFS将不同的各层目录进行合并，但是Upper Dir和各Lower Dir这几个不同的目录并不完全等价，存在层次关系。

- 首先当Upper Dir和Lower Dir存在同名文件时，Lower Dir的文件将会被隐藏，用户只能看见来自Upper Dir的文件
- 各个Lower Dir也存在层次关系，较上层屏蔽较下层的同名文件
- 如果存在同名目录，那么Lower Dir和Upper Dir目录中的内容将会合并
- 当用户修改Merge Dir中来自Upper Dir的数据时，数据将直接写入Upper Dir中原来目录，删除文件也同理
- 当用户修改Merge Dir中来自Lower Dir的数据时，Lower Dir中内容均不会发生任何改变。因为Lower Dir是只读的，用户想修改来自Lower Dir数据时，OverlayFS会首先拷贝一份Lower Dir中文件副本到Upper Dir中。后续修改或删除将会在Upper Dir下的副本中进行，Lower Dir中原文件将会被隐藏

## 实操

先看下OverlayFS的挂载。

```bash
mount -t overlay -o <options> overlay <mount point>
```

`<mount point>`是最终overlay的挂载点。

其中overlay的options有如下：

- lowerdir=<dir>，指定用户需要挂载的lower层目录，lower层支持多个目录，用“:”间隔，优先级依次降低。最多支持500层。
- upperdir=<dir>，指定用户需要挂载的upper层目录，upper层优先级高于所有的lower层目录。
- workdir=<dir>，指定文件系统挂载后用于存放临时和间接文件的工作基础目录。
- default_permissions
- redirect_dir=on/off：开启或关闭redirect directory特性，开启后可支持merged目录和纯lower层目录的rename/renameat系统调用。
- index=on/off：开启或关闭index特性，开启后可避免hardlink copyup broken问题。



```bash
# root @ localhost in ~/playground/test_overlay [0:40:35]
$ ls
lower  merge  upper  work

```

我们建了如上几个目录，然后

```
# root @ localhost in ~/playground/test_overlay [0:40:18]
$ mount -t overlay -o lowerdir=lower,upperdir=upper,workdir=work overlay merge

```

挂载一个OverlayFS文件系统，挂载点为目录`merge`。

```bash
# root @ localhost in ~/playground/test_overlay [1:20:27]
$ echo "i'm file:f1 from lower" > lower/f1.txt

# root @ localhost in ~/playground/test_overlay [1:20:54]
$ echo "i'm file:f2 from lower" > lower/f2.txt

# root @ localhost in ~/playground/test_overlay [1:21:36]
$ echo "i'm file:f3 from upper" > upper/f3.txt

# root @ localhost in ~/playground/test_overlay [1:21:52]
$ tree
.
├── lower
│   ├── f1.txt
│   └── f2.txt
├── merge
│   ├── f1.txt
│   ├── f2.txt
│   └── f3.txt
├── upper
│   └── f3.txt
└── work
    └── work

5 directories, 6 files
```

我们在lower目录创建了f1.txt和f2.txt两个文件，在upper目录创建了f3.txt文件。

我们在merge目录看到了f1.txt, f2.txt和f3.txt 这三个文件，其中f1.txt和f2.txt来自lower目录，f3.txt来自upper目录。

我们在merge目录对文件f3.txt的修改，实际上就是对upper目录中f3.txt文件的修改。

```bash
# root @ localhost in ~/playground/test_overlay/merge [13:22:21]
$ echo "changes were made from merge dir" >> f3.txt

# root @ localhost in ~/playground/test_overlay/merge [13:23:19]
$ cat ../upper/f3.txt
i'm file:f3 from upper
changes were made from merge dir

# root @ localhost in ~/playground/test_overlay/merge [13:23:23]
$
```



我们在merge目录对文件f2.txt的修改，OverlayFS会将lower目录中的f2.txt拷贝到upper目录，并对其做修改。lower目录中的f2.txt不会改变。此时，upper目录中的f2.txt会覆盖lower目录的f2.txt，我们再merge目录看到的f2.txt实际上就是upper目录中的f2.txt文件。

```bash
# root @ localhost in ~/playground/test_overlay/merge [13:24:27]
$ echo "changes were made from merge dir" >> f2.txt

# root @ localhost in ~/playground/test_overlay/merge [13:24:43]
$ cat ../lower/f2.txt
i'm file:f2 from lower

# root @ localhost in ~/playground/test_overlay/merge [13:24:50]
$ ls -l ../upper/
总用量 8
-rw-r--r--. 1 root root 56 7月  26 13:24 f2.txt
-rw-r--r--. 1 root root 56 7月  26 13:23 f3.txt

# root @ localhost in ~/playground/test_overlay/merge [13:25:03]
$ cat ../upper/f2.txt
i'm file:f2 from lower
changes were made from merge dir

# root @ localhost in ~/playground/test_overlay/merge [13:25:08]
$
```





![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/overlayfs2.png)