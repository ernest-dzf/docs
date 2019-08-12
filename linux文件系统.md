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

我们以机械硬盘为例。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/chs_address.png)

每个盘面都被划分为数目相等的磁道，并从外缘的“0”开始编号，具有相同编号的磁道形成一个圆柱，称之为磁盘的柱面（C）。磁盘的柱面数与一个盘面上的磁道数是相等的。

由于每个盘面都有自己的磁头，因此，盘面数等于总的磁头数（H）。

机械硬盘其实是通过CHS去寻址的。C-表示柱面，H-表示磁头，S-表示扇区。如果我们知道一个文件对应在硬盘上存储位置的CHS，那么我们就知道这个文件存储在哪里，也可以去对应的位置读取了。

## VFS

VFS在系统调用和具体的文件系统实现（比如xfs，ext2，ext3，ext4）之间提供了一层抽象，为所有的设备提供了统一的接口。
![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/vfs.png)



更详细的图如下，可以看到VFS处于整个linux storage 系统的位置。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/linux_storage_stack.png)

VFS提供了一些系统调用，比如read，write，open，……。

Application不需要知道底层的具体文件系统是什么，他面对的接口都是统一的。就好比你代码里面调用read函数，你不会去case文件是位于ext2、ext3还是ext4文件系统上。