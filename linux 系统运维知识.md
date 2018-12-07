# linux 系统运维知识 #
## /etc/fstab ##

磁盘被手动挂载之后都必须把挂载信息写入/etc/fstab这个文件中，否则下次开机启动时仍然需要重新挂载。

系统开机时会主动读取/etc/fstab这个文件中的内容，根据文件里面的配置挂载磁盘。这样我们只需要将磁盘的挂载信息写入这个文件中我们就不需要每次开机启动之后手动进行挂载了。

### 挂载的限制 ###

1. 根目录是必须挂载的，而且一定要先于其他mount point被挂载。因为mount是所有目录的跟目录，其他木有都是由根目录 /衍生出来的。
2. 挂载点必须是已经存在的目录。
3. 挂载点的指定可以任意，但必须遵守必要的系统目录架构原则。
4. 所有挂载点在同一时间只能被挂载一次。
5. 所有分区在同一时间只能挂在一次。
6. 若进行卸载，必须将工作目录退出挂载点（及其子目录）之外。

### /etc/fstab文件中的参数 ###

先看下`/etc/fstab`的内容

	[root@TENCENT64 ~]# cat /etc/fstab 
	#
	# /etc/fstab
	# Created by anaconda
	#
	# Accessible filesystems, by reference, are maintained under '/dev/disk'
	# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
	#
	/dev/sda1               /                       ext4    noatime,acl,user_xattr  1 1
	/dev/sda2               swap                    swap    defaults        0 0
	/dev/sda3               /usr/local              ext4    noatime,acl,user_xattr  1 2
	/dev/sda4               /data                   ext4    noatime,acl,user_xattr  1 2
	tmpfs                   /dev/shm                tmpfs   defaults        0 0
	devpts                  /dev/pts                devpts  gid=5,mode=620  0 0
	sysfs                   /sys                    sysfs   defaults        0 0
	proc                    /proc                   proc    defaults        0 0
