# linux文件系统

先来一个感性的概念。

文件系统是位于磁盘上的，一种数据的组织形式。你可以对一个raw disk进行格式化，刷入特定的文件系统；你也可以对某一个分区进行格式化，刷入特定的文件系统。

具体到操作来说，比如：	

```shell
mkfs -t ext4 /dev/nvme0n1
```

一个磁盘或者分区，文件系统刷好之后，就可以进行挂载了（mount），挂载好之后就可以使用了。

如果你把这个刷好文件系统的磁盘umount掉，然后在其他操作系统mount上去，还是可以正常使用的。这里想说的是，文件系统在磁盘上是闭环的，你刷好之后，他就在磁盘上，不依赖其他额外的东西。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/block_group_1.png)

## VFS

VFS在系统调用和具体的文件系统实现（比如xfs，ext2，ext3，ext4）之间提供了一层抽象，为所有的设备提供了统一的接口。
![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/vfs.png)

更