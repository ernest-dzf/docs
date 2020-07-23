# AUFS简介

AUFS - Advanced Union File System，可以用来把不同物理位置的东西合并到同一个目录下面

## 安装

针对CentOS - 7为例，

```bash
cd /etc/yum.repo.d
wget https://yum.spaceduck.org/kernel-ml-aufs/kernel-ml-aufs.repo
yum install kernel-ml-aufs
```

修改内核启动，

```
vi /etc/default/grub
# 修改参数, 表示启动时选择第一个内核
###################################
GRUB_DEFAULT=0
###################################

# 重新生成grub.cfg
grub2-mkconfig -o /boot/grub2/grub.cfg
# 重启计算机
reboot
```

`GRUB_DEFAULT`为`saved`，`saved`表示下次启动时`默认启动上次的内核`，这里我们需要更改`GRUB_DEFAULT=0`, 表示启动时选择第一个内核。

重启之后，查看是否按照成功，

```bash
[root@VM_0_15_centos mnt]# cat /proc/filesystems
nodev	sysfs
nodev	tmpfs
nodev	bdev
nodev	proc
nodev	cgroup
nodev	cgroup2
nodev	cpuset
nodev	devtmpfs
nodev	configfs
nodev	debugfs
nodev	tracefs
nodev	securityfs
nodev	sockfs
nodev	pipefs
nodev	ramfs
nodev	hugetlbfs
nodev	devpts
nodev	autofs
nodev	aufs
nodev	mqueue
nodev	pstore
	ext3
	ext2
	ext4
	iso9660
[root@VM_0_15_centos mnt]#
```

可以看到，已经ok。

## 使用aufs

```bash
[root@VM_0_15_centos ~]# mkdir aufs
[root@VM_0_15_centos ~]# cd aufs/
[root@VM_0_15_centos aufs]# mkdir mnt
[root@VM_0_15_centos aufs]# mkdir container-layer
[root@VM_0_15_centos aufs]# echo "I am container layer" > container-layer/container-layer.txt
[root@VM_0_15_centos aufs]# mkdir ../aufs/{image-layer1,image-layer2,image-layer3}
[root@VM_0_15_centos aufs]# ls
container-layer  image-layer1  image-layer2  image-layer3  mnt
[root@VM_0_15_centos aufs]# echo "I am image layer 1" > ../aufs/image-layer1/image-layer1.txt
[root@VM_0_15_centos aufs]# echo "I am image layer 2" > ../aufs/image-layer2/image-layer2.txt
[root@VM_0_15_centos aufs]# echo "I am image layer 3" > ../aufs/image-layer3/image-layer3.txt
[root@VM_0_15_centos aufs]#
[root@VM_0_15_centos aufs]# tree
.
├── container-layer
│   └── container-layer.txt
├── image-layer1
│   └── image-layer1.txt
├── image-layer2
│   └── image-layer2.txt
├── image-layer3
│   └── image-layer3.txt
└── mnt

5 directories, 4 files
```

按照如上创建一个aufs目录，并在其下面创建子目录和相关文件，用于演示。

我们这里创建了一个`container-layer`目录，用于模拟容器的读写层；创建了`image-layer1`，`image-layer2`，`image-layer3`这三个目录用于模拟容器的镜像层。

同时创建了一个`mnt`目录用于文件系统挂载点。

接下来，我们使用`mount`命令，创建aufs文件系统。

```
[root@VM_0_15_centos aufs]# mount -t aufs -o dirs=./container-layer:./image-layer1:./image-layer2:./image-layer3 none ./mnt
```

挂载ok之后，目录树结构如下，

```bash
[root@VM_0_15_centos aufs]# tree
.
├── container-layer
│   └── container-layer.txt
├── image-layer1
│   └── image-layer1.txt
├── image-layer2
│   └── image-layer2.txt
├── image-layer3
│   └── image-layer3.txt
└── mnt
    ├── container-layer.txt
    ├── image-layer1.txt
    ├── image-layer2.txt
    └── image-layer3.txt

5 directories, 8 files
[root@VM_0_15_centos aufs]#
```

注意，在 mount 命令中，我们没有指定要挂载的 4 个文件夹的权限信息，其默认行为是：dirs 指定的左边起第一个目录是 read-write 权限，后续目录都是 read-only 权限。

我们可以这样查看详情，

```bash
[root@VM_0_15_centos aufs]# cat /sys/fs/aufs/si_98c9f806916ec97d/*
/root/aufs/container-layer=rw
/root/aufs/image-layer1=ro
/root/aufs/image-layer2=ro
/root/aufs/image-layer3=ro
64
65
66
67
/root/aufs/container-layer/.aufs.xino
[root@VM_0_15_centos aufs]#

```

其中的 si_98c9f806916ec97d 目录是系统为 mnt 这个挂载点创建的，从上图中我们可以清楚的看到各个目录的挂载权限。

## 验证写时复制

```bash
[root@VM_0_15_centos aufs]# echo "I changed mnt/image-layer2.txt" >> mnt/image-layer2.txt
[root@VM_0_15_centos aufs]#

```

我们发现，`image-layer2/image-layer2.txt`文件并没有变化，

```
[root@VM_0_15_centos aufs]# cat image-layer2/image-layer2.txt
I am image layer 2
[root@VM_0_15_centos aufs]#

```

相反，`container-layer`目录下多了一个文件，

```bash
[root@VM_0_15_centos container-layer]# ls
container-layer.txt  image-layer2.txt
[root@VM_0_15_centos container-layer]# cat image-layer2.txt
I am image layer 2
I changed mnt/image-layer2.txt
[root@VM_0_15_centos container-layer]#
```

原来，当尝试向 mnt/image-layer2.txt 中写入文件时，系统首先在 mnt 目录下查找名为 image-layer2.txt 的文件，将其拷贝到 read-write 层的 container-layer 目录中，接着对 container-layer 目录中的 image-layer2.txt 的文件进行写操作。这个过程也就是 AUFS 的实际工作原理。

可以看出 AUFS 的原理并不是很复杂，但是当与容器技术相结合后，就展示出来超强的魅力，并成为初代 docker 默认的存储驱动。虽然当前 docker 默认的存储驱动已经演进到了 overlay2，但是学习 AUFS 依然可以帮助我们深入理解 docker 中的文件系统。