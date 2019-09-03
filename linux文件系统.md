# linux文件系统

先来一个感性的概念。

文件系统是位于磁盘上的，一种数据的组织形式。你可以对一个raw disk进行格式化，刷入特定的文件系统；你也可以对某一个分区进行格式化，刷入特定的文件系统。

具体到操作来说，比如：	

```shell
mkfs -t ext4 /dev/nvme0n1
```

一个磁盘或者分区，文件系统刷好之后，就可以进行挂载了（mount），挂载好之后就可以使用了。

如果你把这个刷好文件系统的磁盘umount掉，然后在其他操作系统mount上去，还是可以正常使用的。这里想说的是，文件系统在磁盘上是闭环的，你刷好之后，他就在磁盘上，不依赖其他额外的东西。

我们以机械硬盘为例。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/chs_address.png)

每个盘面都被划分为数目相等的磁道，并从外缘的“0”开始编号，具有相同编号的磁道形成一个圆柱，称之为磁盘的柱面（C，cylinder）。磁盘的柱面数与一个盘面上的磁道数是相等的。

由于每个盘面都有自己的磁头，因此，盘面数等于总的磁头数（H，Head）。

机械硬盘其实是通过CHS去寻址的。C-表示柱面，H-表示磁头，S-表示扇区（Sector）。如果我们知道一个文件对应在硬盘上存储位置的CHS，那么我们就知道这个文件存储在哪里，也可以去对应的位置读取了。

那对于每个文件，我们必须有个地方存储这些地址信息。我们还必须知道哪些地址空间是free的，哪些是已经存了数据的。一个稳定件被删除的话，我们还得更新相关记录，……

总之就会有一堆事情需要干，文件系统可以帮我们做这些繁琐的事情。

## 文件系统的作用

文件系统的作用主要体现在三个方面：

1. **便于磁盘空间的管理**
   想想，如果没有文件系统，对于每个文件，你得记住它放在哪个地址上，大小多大，权限属性是啥。后面删除了的话，你还得更新这段空间是删除了的。总之有一堆事情要干。
   有了文件系统之后，我们不需要考虑这些是怎么存，怎么管理的了。

1. **方便数据的组织和查找**
   先感受下linux下文件的组织形式。

   ```shell
   [root@VM_144_188_centos ~]# tree -L 2
   .
   |-- Changelog
   |-- curl-format.txt
   |-- dmesg.log
   |-- install
   |   `-- monitor_agent_cloud_update-1.0.24-1.0.31-monitor_agent_cloud
   |-- ldap_monitor.sh
   |-- pam_ldap.so
   |-- pprof
   |   |-- pprof.cynosdb_agent.contentions.delay.001.pb.gz
   |   |-- pprof.cynosdb_agent.samples.cpu.001.pb.gz
   |   |-- pprof.cynosdb_agent.samples.cpu.002.pb.gz
   |   `-- pprof.cynosdb_agent.samples.cpu.003.pb.gz
   |-- services
   |   |-- ars_tsc_tools -> /usr/local/services/ars_tsc_tools-1.0
   |   |-- cloud_clear_disk -> /usr/local/services/cloud_clear_disk-1.0
   |   |-- cloud_core_check -> /usr/local/services/cloud_core_check-1.0
   |   `-- monitor_agent_cloud -> /usr/local/services/monitor_agent_cloud-1.0
   |-- set_etcd.txt
   `-- softinst.sh
   
   8 directories, 11 files
   [root@VM_144_188_centos ~]# 
   
   ```

   我们将所有数据组织成非常有条理的树形结构，使我们对数据有了很清晰的规划，也很方便后续查找我们想要的数据

2. **提高磁盘空间的使用率**
   如果没有文件系统，存储在磁盘中的数据，经过多次增删改查之后，会出现空间空洞。这些空洞是碎片化的，不能充分利用。比如你要存储一个大小为2G的文件，但是目前磁盘中最大的连续空间只有1.5G，虽然总的剩余空间有几十个G，但你还是不能存下这个2G的文件。
   文件系统会对空洞和数据进行交换，从而生成比较大块的可用磁盘空间。





总结一下记住这两点：

- 文件系统实际上就是数据的结构，数据在磁盘是如何组织的。了解文件系统就是了解磁盘上的那么多bytes是如何被解释的。为啥这段bytes就表示文件名？为啥那段bytes就表示的是数据？为啥这段bytes表示的就是文件被修改的时间？……等等这些问题需要一个规范，让我们可以依据这个规范去解释这些bytes。bytes本身是无意义的，只有按照一定的规则去解释的时候，才有具体的含义。
- 实现一个文件系统就是设计一种机制，让一块磁盘中的bytes可以被正确解释为诸如文件、目录或者其他结构。你用代码去实现了这种机制，你就实现了文件系统。



## ext2文件系统布局

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/block_group_1.png)



### block

文件系统的block的大小是在格式化的时候就确定了，也就是`mkfs`的时候就确定了。对于一个已经格式化好的文件系统，可以通过`tune2fs`来查看block 的size。

```shell
[root@localhost ~]# tune2fs  -l /dev/md0p1 |grep "Block size"
Block size:               4096
[root@localhost ~]# 

	
```

可以看到`/dev/md0p1`的block的大小是4096字节。

### boot block

boot block的大小是确定的，就是1024字节。

这是为分区的引导扇区保留的，任何文件系统都不能使用启动块，你可以不用关注boo block。

### dumpe2fs

可以利用`dumpe2fs`获取ext2/3/4文件系统的信息，诸如superblock、group descriptors等信息。

### superblock

下面是内核中superblock结构体的定义。

```c
struct ext2_super_block {
	__le32	s_inodes_count;		/* Inodes count */
	__le32	s_blocks_count;		/* Blocks count */
	__le32	s_r_blocks_count;	/* Reserved blocks count */
	__le32	s_free_blocks_count;	/* Free blocks count */
	__le32	s_free_inodes_count;	/* Free inodes count */
	__le32	s_first_data_block;	/* First Data Block */
	__le32	s_log_block_size;	/* Block size */
	__le32	s_log_frag_size;	/* Fragment size */
	__le32	s_blocks_per_group;	/* # Blocks per group */
	__le32	s_frags_per_group;	/* # Fragments per group */
	__le32	s_inodes_per_group;	/* # Inodes per group */
	__le32	s_mtime;		/* Mount time */
	__le32	s_wtime;		/* Write time */
	__le16	s_mnt_count;		/* Mount count */
	__le16	s_max_mnt_count;	/* Maximal mount count */
	__le16	s_magic;		/* Magic signature */
	__le16	s_state;		/* File system state */
	__le16	s_errors;		/* Behaviour when detecting errors */
	__le16	s_minor_rev_level; 	/* minor revision level */
	__le32	s_lastcheck;		/* time of last check */
	__le32	s_checkinterval;	/* max. time between checks */
	__le32	s_creator_os;		/* OS */
	__le32	s_rev_level;		/* Revision level */
	__le16	s_def_resuid;		/* Default uid for reserved blocks */
	__le16	s_defq_resgid;		/* Default gid for reserved blocks */
	/*
	 * These fields are for EXT2_DYNAMIC_REV superblocks only.
	 *
	 * Note: the difference between the compatible feature set and
	 * the incompatible feature set is that if there is a bit set
	 * in the incompatible feature set that the kernel doesn't
	 * know about, it should refuse to mount the filesystem.
	 * 
	 * e2fsck's requirements are more strict; if it doesn't know
	 * about a feature in either the compatible or incompatible
	 * feature set, it must abort and not try to meddle with
	 * things it doesn't understand...
	 */
	__le32	s_first_ino; 		/* First non-reserved inode */
	__le16   s_inode_size; 		/* size of inode structure */
	__le16	s_block_group_nr; 	/* block group # of this superblock */
	__le32	s_feature_compat; 	/* compatible feature set */
	__le32	s_feature_incompat; 	/* incompatible feature set */
	__le32	s_feature_ro_compat; 	/* readonly-compatible feature set */
	__u8	s_uuid[16];		/* 128-bit uuid for volume */
	char	s_volume_name[16]; 	/* volume name */
	char	s_last_mounted[64]; 	/* directory where last mounted */
	__le32	s_algorithm_usage_bitmap; /* For compression */
	/*
	 * Performance hints.  Directory preallocation should only
	 * happen if the EXT2_COMPAT_PREALLOC flag is on.
	 */
	__u8	s_prealloc_blocks;	/* Nr of blocks to try to preallocate*/
	__u8	s_prealloc_dir_blocks;	/* Nr to preallocate for dirs */
	__u16	s_padding1;
	/*
	 * Journaling support valid if EXT3_FEATURE_COMPAT_HAS_JOURNAL set.
	 */
	__u8	s_journal_uuid[16];	/* uuid of journal superblock */
	__u32	s_journal_inum;		/* inode number of journal file */
	__u32	s_journal_dev;		/* device number of journal file */
	__u32	s_last_orphan;		/* start of list of inodes to delete */
	__u32	s_hash_seed[4];		/* HTREE hash seed */
	__u8	s_def_hash_version;	/* Default hash version to use */
	__u8	s_reserved_char_pad;
	__u16	s_reserved_word_pad;
	__le32	s_default_mount_opts;
 	__le32	s_first_meta_bg; 	/* First metablock block group */
	__u32	s_reserved[190];	/* Padding to the end of the block */
};
```



superblock总是位于偏移量1024字节开始，占用空间也是1024字节。

如果一个block的大小大于1024字节的话，superblock也只占用1024字节，后面的会补齐。也就是说在superblock和group descriptors之间会有padding。

使用hexdump解析下superblock。

```shell
hexdump -s 1024 -n 52 -e '1/4 "%-15d" 3/4 " %-15d" "\n"' /dev/sdc |less	
```

可以使用`e2label`修改文件系统的label。

> e2label will display or change the filesystem label on the ext2, ext3, or ext4 filesystem

```shell
[root@localhost data6]# e2label /dev/sdc victor

```



## VFS

VFS在系统调用和具体的文件系统实现（比如xfs，ext2，ext3，ext4）之间提供了一层抽象，为所有的设备提供了统一的接口。
![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/vfs.png)



更详细的图如下，可以看到VFS处于整个linux storage 系统的位置。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/linux_storage_stack.png)

VFS提供了一些系统调用，比如read，write，open，……。

Application不需要知道底层的具体文件系统是什么，他面对的接口都是统一的。就好比你代码里面调用read函数，你不会去case文件是位于ext2、ext3还是ext4文件系统上。