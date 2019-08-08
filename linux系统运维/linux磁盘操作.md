# linux磁盘操作

## lsblk

lsblk是一个常用的命令，可以展示机器所有的block devices，包括已挂载和未挂载的。

```shell
[root@localhost ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0              11:0    1  4.3G  0 rom  
nvme0n1         259:0    0   20G  0 disk 
├─nvme0n1p1     259:1    0    1G  0 part /boot
└─nvme0n1p2     259:2    0   19G  0 part 
  ├─centos-root 253:0    0   17G  0 lvm  /
  └─centos-swap 253:1    0    2G  0 lvm  [SWAP]
nvme0n2         259:3    0   20G  0 disk 
nvme0n3         259:4    0   20G  0 disk 
```

没有显示挂载点的设备就是没有挂载的，需要挂载之后才可以访问。

比如上面，`mount /dev/sr0 /mnt`之后，结果如下：

```shell
[root@localhost /]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0              11:0    1  4.3G  0 rom  /mnt
nvme0n1         259:0    0   20G  0 disk 
├─nvme0n1p1     259:1    0    1G  0 part /boot
└─nvme0n1p2     259:2    0   19G  0 part 
  ├─centos-root 253:0    0   17G  0 lvm  /
  └─centos-swap 253:1    0    2G  0 lvm  [SWAP]
nvme0n2         259:3    0   20G  0 disk 
nvme0n3         259:4    0   20G  0 disk 
[root@localhost /]# 
```

挂载之后当然也可以卸载。

```
umount /dev/sr0
```

或者

```
umount /mnt
```

都可以卸载sr0设备。



## nvme0,nvme0n1,nvme0n1p1

还是以上面例子为例，nvme0、nvme0n1、nvme0n1p1分别表示什么呢？

- nvme0: first registered device's device controller
- nvme0n1: first registered device's first namespace，可以理解为我们使用sas盘时看到的`/dev/sd[abc]`
- nvme0n1p1: first registered device's first namespace's first
  partition，可以理解为我们使用sas盘时看到的`/dev/sda[1234]`



## nvme0n2

lsblk还可以看到nvme0n2以及nvme0n3，这些又都是什么呢？

nvme0n2和nvme0n3是这台机器的另外两个nvme盘，大小都是20G。他们没有挂载，也没有分区。

```shell
[root@localhost /]# mount /dev/nvme0n3 /data3/
mount: /dev/nvme0n3 写保护，将以只读方式挂载
mount: 未知的文件系统类型“(null)”
[root@localhost /]# 
```

我们直接挂载nvme0n3，报错了。原因是我们还没有格式化nvme0n3盘，没有文件系统呢。

```shell
[root@localhost /]# mkfs.xfs /dev/nvme0n3 
meta-data=/dev/nvme0n3           isize=512    agcount=4, agsize=1310720 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=5242880, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@localhost /]# mount /dev/nvme0n3 /data3/
[root@localhost /]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0              11:0    1  4.3G  0 rom  
nvme0n1         259:0    0   20G  0 disk 
├─nvme0n1p1     259:1    0    1G  0 part /boot
└─nvme0n1p2     259:2    0   19G  0 part 
  ├─centos-root 253:0    0   17G  0 lvm  /
  └─centos-swap 253:1    0    2G  0 lvm  [SWAP]
nvme0n2         259:3    0   20G  0 disk /data2
nvme0n3         259:4    0   20G  0 disk /data3
[root@localhost /]# 

```

先利用`mkfs.xfs`格式化一下nvme0n3盘，然后挂载成功。

总结：**需要先有文件系统，才可以挂载**。

## 重启后挂载又消失了

还是刚才的例子，我们reboot之后，发现nvme0n2和nvme0n3并没有挂载在/data2和/data3目录下。

```shell
[root@localhost ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0              11:0    1  4.3G  0 rom  
nvme0n1         259:0    0   20G  0 disk 
├─nvme0n1p1     259:1    0    1G  0 part /boot
└─nvme0n1p2     259:2    0   19G  0 part 
  ├─centos-root 253:0    0   17G  0 lvm  /
  └─centos-swap 253:1    0    2G  0 lvm  [SWAP]
nvme0n2         259:3    0   20G  0 disk 
nvme0n3         259:4    0   20G  0 disk 
[root@localhost ~]# 
```

也就是我们刚刚做的更改并不是永久的，重启系统之后，又需要我们重新挂载（格式化不需要我们重新做了，只需要重新挂载）。

有没有什么办法解决了？

通过`/etc/fstab`解决。



## /etc/fstab

`/etc/fstab`是系统启动时候自动挂载设备的依据。

```shell
[root@localhost ~]# cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Sat Jul 27 20:28:59 2019
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=832ba9de-ebc7-4263-837a-d8e099bead17 /boot                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
[root@localhost ~]# 


```

上面是`/etc/fstab`文件的一个例子。每一行确定一个挂载项目。每行分6列。



- 第一列：分区名（设备ID）。

  第一列写明你要挂载哪个设备。你可以写设备名（比如`/dev/nvme0n2`)，也可以写设备的uuid，还可以写设备的卷标（卷标好像只能在ext3/ext4文件系统使用，参考[这里](https://www.cyberciti.biz/faq/rhel-centos-debian-fedora-mount-partition-label/)）。

  设备的uuid是设备的属性，和设备挂载在什么系统，什么目录没有关系。同一个设备的uuid 是固定的。使用uuid挂载的好处是，确保同一个设备始终是被挂载在同一个目录上。	

  我们不使用设备名称挂载是有原因的，因为设备名称并非总是一致的，它依赖于启动时内核加载模块的顺序。比如你第一次插了一个u盘启动了系统，下一次又把它拔掉了，这可能导致其他设备的设备名称出现变化。

  如何查看uuid呢？

  - blkid
  - ls -l /dev/disk/by-uuid

  ```shell
  [root@localhost ~]# blkid
  /dev/nvme0n1p1: UUID="832ba9de-ebc7-4263-837a-d8e099bead17" TYPE="xfs" 
  /dev/nvme0n1p2: UUID="aR9beT-s2ZS-ty2e-dylv-z2Wb-EwJI-1YXMY5" TYPE="LVM2_member" 
  /dev/nvme0n2: UUID="458d420c-6f35-4ef1-b14b-281dadcf515b" TYPE="xfs" 
  /dev/nvme0n3: UUID="373bec2c-ee13-489b-8ec8-597146add899" TYPE="xfs" 
  /dev/sr0: UUID="2018-11-25-23-54-16-00" LABEL="CentOS 7 x86_64" TYPE="iso9660" PTTYPE="dos" 
  /dev/mapper/centos-root: UUID="568eb622-3c00-416f-9bc6-af06b8493457" TYPE="xfs" 
  /dev/mapper/centos-swap: UUID="32e35f4d-8006-40f9-ad05-c016bc4fb773" TYPE="swap" 
  /dev/nvme0n1: PTTYPE="dos" 
  [root@localhost ~]# ls -l /dev/disk/by-uuid
  总用量 0
  lrwxrwxrwx. 1 root root  9 7月  29 01:55 2018-11-25-23-54-16-00 -> ../../sr0
  lrwxrwxrwx. 1 root root 10 7月  29 01:55 32e35f4d-8006-40f9-ad05-c016bc4fb773 -> ../../dm-1
  lrwxrwxrwx. 1 root root 13 7月  29 01:55 373bec2c-ee13-489b-8ec8-597146add899 -> ../../nvme0n3
  lrwxrwxrwx. 1 root root 13 7月  29 01:55 458d420c-6f35-4ef1-b14b-281dadcf515b -> ../../nvme0n2
  lrwxrwxrwx. 1 root root 10 7月  29 01:55 568eb622-3c00-416f-9bc6-af06b8493457 -> ../../dm-0
  lrwxrwxrwx. 1 root root 15 7月  29 01:55 832ba9de-ebc7-4263-837a-d8e099bead17 -> ../../nvme0n1p1
  [root@localhost ~]# 
  
  
  
  ```

  

- 第二列：挂载点

  挂载点必须为当前已经存在的目录。对于交换分区(swap)，这个字段定义为swap，如果在载入点的路径中包含空格符，可以用“/040”来替代空格符。

- 第三列：文件系统类型

  Linux系统支持大量的文件类型，常见的有：ext2、ext3、ext4、xfs(CentOS7)、iso9660(光盘)等，此字段须与分区格式化时使用的类型相同。也可以使用  auto  这一特殊的语法，使系统自动侦测目标分区的分区类型。auto通常用于可移动设备的挂载。如果想了解你的系统目前支持哪些文件系统，可以查看/proc/filesystems的内容。

- 第四列：挂载参数

  指定加载该设备的文件系统时需要使用的特定参数选项，多个参数是由逗号分隔开来。常见参数如下：

  - auto：系统自动挂载，fstab默认就是这个选项
  - defaults：最常见参数，可以满足大多数文件系统使用
  - noauto：开机不自动挂载
  - nouser：只有超级用户可以挂载
  - ro：按只读权限挂载
  - rw：按可读可写权限挂载
  - user：任何用户都可以挂载

- 第五列：dump备份设置

  当其值设置为1时，将允许dump备份程序备份；设置为0时，忽略备份操作。	

- 第六列：开机磁盘检查顺序

  数字越小越优先检查，如果两个分区的数字相同，则同时检查。当其值为0时，永远不检查。根”/”分区永远都为1。其它分区从2开始。



我们修改`/etc/fstab`文件，添加nvme0n2和nvme0n3的自动挂载，如下：

```shell
[root@localhost ~]# cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Sat Jul 27 20:28:59 2019
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=832ba9de-ebc7-4263-837a-d8e099bead17 /boot                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
UUID=458d420c-6f35-4ef1-b14b-281dadcf515b /data2 xfs    defaults        0 0
UUID=373bec2c-ee13-489b-8ec8-597146add899 /data3 xfs    defaults        0 0
[root@localhost ~]# 


	
```

这样重启之后，nvme0n2和nvme0n3依然是被挂载的。

## 磁盘分区

一块物理硬盘，可以划分成多个分区，分区信息存放在分区表里。

分区表定义与保存了硬盘的分区信息，分区表位于硬盘开头的一段特定的物理空间内，操作系统等软件通过读取分区表内的信息，就能够获得该硬盘的分区信息。

[参考](https://blog.csdn.net/for_cxc/article/details/88984733)

可以通过`fdisk -l`获取磁盘分区信息，比如是gpt分区还是mbr分区，比如每个分区的大小，……

下面是`fdisk -l`输出的一个例子：

```shell
磁盘 /dev/nvme0n1：21.5 GB, 21474836480 字节，41943040 个扇区                                                
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x000ac222

        设备 Boot      Start         End      Blocks   Id  System                                            
/dev/nvme0n1p1   *        2048     2099199     1048576   83  Linux                                           
/dev/nvme0n1p2         2099200    41943039    19921920   8e  Linux LVM                                       

磁盘 /dev/nvme0n2：21.5 GB, 21474836480 字节，41943040 个扇区                                                
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：gpt
Disk identifier: 48D88E56-C3CA-46F0-94D4-298BADA3362D


#         Start          End    Size  Type            Name                                                   
 1         2048      9764863    4.7G  Microsoft basic v1                                                     
 2      9764864     41940991   15.4G  Microsoft basic v2                                                     

磁盘 /dev/nvme0n3：21.5 GB, 21474836480 字节，41943040 个扇区                                                
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：gpt
Disk identifier: CAFB6AA0-2DE3-4592-86B0-6029236673DE


#         Start          End    Size  Type            Name                                                   
 1         2048      9764863    4.7G  Microsoft basic v3                                                     

磁盘 /dev/mapper/centos-root：18.2 GB, 18249416704 字节，35643392 个扇区                                     
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节


磁盘 /dev/mapper/centos-swap：2147 MB, 2147483648 字节，4194304 个扇区                                       
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节


```

可以看到有三块盘，分表是`nvme0n1`，`nvme0n2`，`nvme0n3`。

#### nvme0n1

其中`nvme0n1`采用的是mbr分区（硬盘标签类型为dos），`nvme0n2`和`nvme0n3`都是采用的gpt分区。

`nvme0n1`分了两个分区。

第一分区从2048扇区起始，至2099199扇区结束。总共2099199-2048+1=2097152个扇区。block共有1048576个，每个block两个扇区。每个扇区512字节。

第二分区从2099200扇区开始，至41943039扇区结束，总共41943039-2099200+1=39843840个扇区。block共有19921920个，每个block两个扇区。每个扇区512字节。

我们还可以看到扇区[0, 2047]是没有被分区的。

这其中，第0号扇区为主引导扇区，1～2047号扇区用于存储grub stage1.5。

在传统的MBR分区方案中，第0号扇区的的[0, 511]个字节存储的是引导程序，用于服务器启动时引导BIOS的加电自检以及GRUB stage1的加载。

而446~509的这64个字节存放的是分区表信息，其中每个分区占用16字节，因此最多只能包含4个分区的信息。

若想要创建超过5个分区，则需要将其中一个分区转换成逻辑分区，再对逻辑分区进行划分。最后多出来的510～511字节按惯例为0xAA55。若磁盘此处的值不为0xAA55,则判断该磁盘的MBR被损坏。

我们可以通过 hexdump 命令输出MBR中分区表的内容。

```shell
[root@localhost data2_1]# hexdump -s 446 -n 66 -e '1/1 "%02x" " " 3/1 "%02x" " " 1/1 "%02x" " " 3/1 "%02x" " " 2/4 "%10d" "\n"' /dev/nvme0n1
80 202100 83 aa2882       2048   2097152
00 aa2982 8e feffff    2099200  39843840
00 000000 00 000000          0         0
*
55 aa                                   
[root@localhost data2_1]# 

```

由于每个分区只有16个字节，因此能记录的信息十分有限，只包括引导标志、分区格式ID、用CHS(磁头、柱面、扇区)方式描述的分区开始位置和结束位置、用LBA方式(逻辑块)描述的分区开始位置以及包含的扇区数，这些信息。

从中可以看出，这个硬盘有两个分区(第三个分区信息全是0)，其中：

- 第0个字节是引导标志，80表示引导，也就是系统从第1块分区引导
- 第1～3个字节是CHS方式表示的起始位置，目前CHS表示法已经被淘汰了，没有意义
- 第4个字节是分区类型ID号
- 第5～7个字节是CHS方式表示的结束位置,目前CHS表示法已经被淘汰了，没有意义
- 第8～11个字节是LBA方式描述的起始扇区号，也就是2048，与之前通过`fdisk -l`获取的结果一样
- 第12～15个字节是LBA方式描述的扇区数，也就是2097152，与之前通过`fdisk -l`获取的结果一样

#### nvme0n2

`nvme0n2`有两个分区。

第一个分区从2048扇区起始，至9764863扇区结束。

第二个分区从9764864扇区起始，至41940991扇区结束。

`nvme0n2`采用的是gpt分区。

#### nvme0n3

`nvme0n3`有1个分区。分区从2048扇区起始，至9764863扇区结束。



## gpt分区与mbr分区

#### gpt分区



#### mbr分区

## 分区操作

### 创建分区

我们可以使用`parted`工具来对磁盘进行分区。具体操作分为如下几步：

1. `select /dev/sdb`
2. `mklable gpt`
3. `mkpart`
4. `quit`

上面是使用`parted`交互式分区的步骤。



```shell
[root@localhost ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   20G  0 disk 
├─sda1   8:1    0    1G  0 part /boot
├─sda2   8:2    0    2G  0 part [SWAP]
└─sda3   8:3    0   17G  0 part /
sdb      8:16   0   20G  0 disk 
├─sdb1   8:17   0  4.7G  0 part 
└─sdb2   8:18   0  4.7G  0 part 
sdc      8:32   0   20G  0 disk 
sr0     11:0    1 1024M  0 rom  
[root@localhost ~]# parted
GNU Parted 3.1
使用 /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) select /dev/sdc
使用 /dev/sdc
(parted) mklabel gpt
(parted) mkpart                                                           
分区名称？  []? c1                                                        
文件系统类型？  [ext2]? ext4                                              
起始点？ 1                                                                
结束点？ 6000                                                             
(parted) print                                                            
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdc: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name  标志
 1      1049kB  6000MB  5999MB               c1

(parted) quit                                                             
信息: You may need to update /etc/fstab.

[root@localhost ~]#                                                       

[root@localhost ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   20G  0 disk 
├─sda1   8:1    0    1G  0 part /boot
├─sda2   8:2    0    2G  0 part [SWAP]
└─sda3   8:3    0   17G  0 part /
sdb      8:16   0   20G  0 disk 
├─sdb1   8:17   0  4.7G  0 part 
└─sdb2   8:18   0  4.7G  0 part 
sdc      8:32   0   20G  0 disk 
└─sdc1   8:33   0  5.6G  0 part 
sr0     11:0    1 1024M  0 rom  
[root@localhost ~]# 

```

上面是使用`parted`交互式命令进行gpt分区的步骤。

最终结果是对磁盘`sdc`进行分区，得到了一个5.6G大小的分区，分区文件系统是ext4。

使用`fdisk -l`获取得到的sdc的信息如下：

```shell
磁盘 /dev/sdc：21.5 GB, 21474836480 字节，41943040 个扇区                                                    
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：gpt
Disk identifier: B836E489-C9FA-4A76-94B1-65B0E5821DFD


#         Start          End    Size  Type            Name                                                   
 1         2048     11718655    5.6G  Microsoft basic c1                                                     
(END)

```

**文件系统是建立在分区之上的**。分区做好之后，我们也可以利用`mkfs`改变分区的文件系统。

### 删除分区

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/rmpart.png)

利用parted工具，可以删除分区。

### 格式化分区

可以使用`mkfs.ext3 /dev/sdc1`来格式化分区，或者`mkfs -s ext4 /dev/sdc1`这样也行。

## raid

### raid介绍

RAID （ Redundant Array of Independent Disks ）即独立磁盘冗余阵列，通常简称为磁盘阵列。

RAID 主要利用数据条带、镜像和数据校验技术来获取高性能、可靠性、容错能力和扩展性，根据运用或组合运用这三种技术的策略和架构，可以把 RAID 分为不同的等级，以满足不同数据应用的需求。

目前应用最多的是RAID0 、 RAID1 、 RAID3 、 RAID5 、 RAID6 和 RAID10。

从实现角度看， RAID 主要分为**软 RAID**、**硬 RAID** 以及**软硬混合** RAID 三种。

软 RAID 所有功能均由操作系统和 CPU 来完成，没有独立的 RAID 控制 / 处理芯片和 I/O 处理芯片，效率自然最低。

硬 RAID 配备了专门的 RAID 控制 / 处理芯片和 I/O 处理芯片以及阵列缓冲，不占用 CPU 资源，但成本很高。

软硬混合 RAID 具备 RAID 控制 / 处理芯片，但缺乏 I/O 处理芯片，需要 CPU 和驱动程序来完成，性能和成本 在软 RAID 和硬 RAID 之间。

#### raid0

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/raid0.png)

raid0是一种无数据校验的数据条带化技术。它没有数据冗余，将数据分散存储在不同的磁盘上，达到提高io的目的。

raid0一般用于对性能要求严格，但对数据安全性和可靠性不高的应用，比如音频、视频，临时数据缓存空间等。

#### raid1

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/raid1.png)

raid1实际上就是镜像，一份数据存两份，磁盘空间利用率50%。

RAID1 应用于对顺序读写性能要求高以及对数据保护极为重视的应用。

#### raid5

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/raid5.png)

RAID5 应该是目前最常见的 RAID 等级。它有数据校验，并将校验数据分布在阵列中所有磁盘。raid5还具备很好的扩展性。当阵列磁盘 数量增加时，并行操作量的能力也随之增长。

上图为例，A<sub>p</sub>，B<sub>p</sub>，C<sub>p</sub>，D<sub>p</sub>就是校验数据，分布在不同磁盘里。即使有一块盘坏了，也不会丢数据，可以通过校验数据恢复出来。

### 做raid

在创建raid之前，磁盘必须要有分区，先利用parted工具对磁盘进行分区。

### 删除raid

1. `cat /proc/mdstat`看下是否有raid在运行
2. 如果有的话，看raid是否挂载了。这个可以通过`df -h`查看。如果有挂载，那么就卸载阵列。`umount /dev/mdxxxxxx`
3. 再停止raid。`mdadm -S /dev/mdxxxxxxx`
4. 删除磁盘，`mdadm --misc --zero-superblock /dev/sd[bcdefghij]`
5. 删除配置文件，一般是在`/etc/mdadm.conf`。最好查看下`/etc/fstab`，看是否有自动挂载。有的话也要删除


## lvm

### lvm简介

LVM是Linux操作系统的逻辑卷管理器。

逻辑卷管理提供了比传统的磁盘和分区视图更高级别的计算机系统上磁盘存储的视图。 这使得系统管理员可以更灵活地将存储分配给应用程序和用户。

举个例子，我们在装系统的时候，是要规划好各个分区的大小的，比如`/boot`分区，`/`分区，`swap`分区。如果后面某个分区空间不够了怎么办呢？你可能会想到再挂一个大磁盘上去，把文件拷过来，给分区 瘦身。

如果你的分区是基于逻辑卷（LVM）的话，那么这个操作就很简单了。

先来看下lvm的架构。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/lvm.png)

上面这张图中，两个物理磁盘以及另外一个物理磁盘的某个分区，被组合成了一个**Volume Group**。

然后有两个**Logical Volume**被创建出来。这两个Logical Volume都是从Volume Group中被创建出来的。

再基于这两个Logical Volume，创建出了对应的文件系统。

这里涉及到几个概念。

### Physical Hard Drive

Physical Hard Drive，就是我们的磁盘，存在形式就是我们安装了一个磁盘之后，在`/dev/`目录下面会有类似`/dev/sda`，`/dev/sdb`这种文件，或者类似`/dev/nvme0n1`，`/dev/nvme0n2`这种。

### Partition

Partition，就是基于我们的磁盘做的分区。基于分区去创建PV的时候，需要设置分区标志为lvm类型（lvm分区id为8e）。前文中提到的`/dev/nvme0n1p2`分区就是lvm类型的。设置方法如下：

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/pvlvm.png)

### Physical Volume

Physical Volume（PV），一个PV典型的就是一个磁盘，也有可能是看起来像一个磁盘的东西，比如做过软raid的设备。上面提到的分区也可以是一个PV，不过需要设置分区标志为lvm（8e）。

在将磁盘或者分区作为PV之前，需要对其进行初始化工作。

使用命令：`pvcreate`

```shell
[root@localhost ~]# pvcreate /dev/nvme0n3p1 
  Physical volume "/dev/nvme0n3p1" successfully created.
[root@localhost ~]# 

```

### Volume Group

Volume Group（VG），卷组是LVM中使用的最高级别的抽象。一个VG包含了多个Physical devices，每个physical device就是上面说的PV。因此，我们可以往一个VG中添加PV，达到扩容VG的目的。

先查看有哪些VG，命令`vgdisplay`。

```shell
[root@localhost ~]# vgdisplay 
  --- Volume group ---
  VG Name               centos
  System ID             
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  7
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               <38.99 GiB
  PE Size               4.00 MiB
  Total PE              9981
  Alloc PE / Size       8959 / <35.00 GiB
  Free  PE / Size       1022 / 3.99 GiB
  VG UUID               JFc6rR-fMF9-4qef-xp2N-ubho-bkz6-UyJnWa
   
[root@localhost ~]# 

```

可以看到有一个VG，VG name是centos。当前这个VG的大小为38.99GiB。我们尝试将上面创建的PV（`/dev/nvme0n3p1`）添加进去，命令`vgextend`。	

```shell
[root@localhost ~]# vgextend centos /dev/nvme0n3
nvme0n3    nvme0n3p1  
[root@localhost ~]# vgextend centos /dev/nvme0n3p1 
  Volume group "centos" successfully extended
[root@localhost ~]# vgdisplay 
  --- Volume group ---
  VG Name               centos
  System ID             
  Format                lvm2
  Metadata Areas        4
  Metadata Sequence No  8
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                4
  Act PV                4
  VG Size               43.64 GiB
  PE Size               4.00 MiB
  Total PE              11172
  Alloc PE / Size       8959 / <35.00 GiB
  Free  PE / Size       2213 / 8.64 GiB
  VG UUID               JFc6rR-fMF9-4qef-xp2N-ubho-bkz6-UyJnWa
   
[root@localhost ~]# 

```

添加完后，可以看到VG的大小为43.64GiB，符合预期。

我们也可以基于以个raw hard disk来创建PV，然后将其加入到VG中。	

```shell
[root@localhost ~]# pvcreate /dev/nvme0n4 
  Physical volume "/dev/nvme0n4" successfully created.
[root@localhost ~]# 
[root@localhost ~]# vgextend centos /dev/nvme0n4 
  Volume group "centos" successfully extended
[root@localhost ~]# 

```

**不推荐**直接基于raw hard disk 创建PV，就算是需要将整块磁盘作为PV，也建议先创建分区，这个分区大小包含整个磁盘，然后基于分区去创建PV。原因是其他OS或者操作工具（比如parted）识别不了LVM的元数据，可能就直接将其作为free的磁盘了，这样很容易被其他工具覆盖写。

具体可以参考[这里](https://unix.stackexchange.com/questions/76588/what-is-the-best-practice-for-adding-disks-in-lvm/76642#76642)。

可以通过`pvdisplay`展示目前有哪些PV。

### Logical Volume

Logical Volume（LV），相当于非LVM系统中的磁盘分区。 LV作为标准块设备可见。因此LV可以包含文件系统。LV是在VG基础上划分出来的。

查看有哪些LV，命令（`lvdisplay`)

```
  [root@localhost ~]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/centos/swap
  LV Name                swap
  VG Name                centos
  LV UUID                KAQeXl-bo1j-eYNP-yC4p-KTcA-ddai-22nLTR                                              
  LV Write Access        read/write
  LV Creation host, time localhost, 2019-07-27 20:28:58 +0800                                                
  LV Status              available
  # open                 2
  LV Size                2.00 GiB
  Current LE             512
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:1

  --- Logical volume ---
  LV Path                /dev/centos/root
  LV Name                root
  VG Name                centos
  LV UUID                83ov8M-wRSs-JRRv-G0g3-Op8Y-nnhl-kg9MOK                                              
  LV Write Access        read/write
  LV Creation host, time localhost, 2019-07-27 20:28:58 +0800                                                
  LV Status              available
  # open                 1
  LV Size                <33.00 GiB
  Current LE             8447
  Segments               3
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0

```

可以看到有两个LV，分别是`/dev/centos/swap`和`/dev/centos/root`。这两个LV分别挂载在`swap`分区和`/`分区上。

```shell
[root@localhost ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0              11:0    1  4.3G  0 rom  
nvme0n1         259:0    0   20G  0 disk 
├─nvme0n1p1     259:1    0    1G  0 part /boot
└─nvme0n1p2     259:2    0   19G  0 part 
  ├─centos-root 253:0    0   33G  0 lvm  /
  └─centos-swap 253:1    0    2G  0 lvm  [SWAP]
nvme0n2         259:3    0   20G  0 disk 
├─nvme0n2p1     259:4    0  4.7G  0 part 
│ └─centos-root 253:0    0   33G  0 lvm  /
└─nvme0n2p2     259:5    0 15.4G  0 part 
  └─centos-root 253:0    0   33G  0 lvm  /
nvme0n3         259:6    0   20G  0 disk 
└─nvme0n3p1     259:7    0  4.7G  0 part 
[root@localhost ~]# 

```

如果我们想扩大根分区`/`的大小呢。对于lvm来说很简单，使用`lvextend`命令即可。	

```shell
[root@localhost ~]# lvextend -r -L +3G /dev/centos/root 
  Size of logical volume centos/root changed from <33.00 GiB (8447 extents) to <36.00 GiB (9215 extents).
  Logical volume centos/root successfully resized.
meta-data=/dev/mapper/centos-root isize=512    agcount=8, agsize=1113856 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=8649728, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 8649728 to 9436160
[root@localhost ~]# 

```

表示扩大`/dev/centos/root`这个LV的大小，增加3G空间。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/lvextend.png)

对比之前`lsblk`的结果，上面截图验证确实增加了3G空间。



### 创建VG

前面谈到了我们如何创建一个PV，然后将这个PV加入到VG，然后扩容LV的操作。

