# mysql 笔记 #
## 安装 ##
这里以centos为例。
### mysql 5.7安装 ###
1. 官网下载linux generic binary包，地址：https://dev.mysql.com/downloads/mysql/
2. 下载得到的是一个xxx.tar.gz的包，比如`mysql-5.7.23-linux-glibc2.12-x86_64.tar.gz`，解压得到`/usr/local/`目录下面去，比如

		# victor @ localhost in /usr/local [8:40:55] 
		$ ls
		bin  etc  games  include  lib  lib64  libexec  mysql  mysql-5.7.23-linux-glibc2.12-x86_64  sbin  share  src
3. 在`/usr/local`目录下建立一个软链接到`mysql-5.7.23-linux-glibc2.12-x86_64`目录

		# victor @ localhost in /usr/local [8:44:20] C:1
		$ ls
		bin  etc  games  include  lib  lib64  libexec  mysql-5.7.23-linux-glibc2.12-x86_64  sbin  share  src
		
		# victor @ localhost in /usr/local [8:44:23] 
		$ sudo ln -s mysql-5.7.23-linux-glibc2.12-x86_64 mysql
		
		# victor @ localhost in /usr/local [8:44:31] 
		$ ls  
		bin  etc  games  include  lib  lib64  libexec  mysql  mysql-5.7.23-linux-glibc2.12-x86_64  sbin  share  src
		
		# victor @ localhost in /usr/local [8:44:36] 
		$ ls -l mysql
		lrwxrwxrwx. 1 root root 35 Oct 10 08:44 mysql -> mysql-5.7.23-linux-glibc2.12-x86_64
4. 安装一些依赖包

		shell> yum search libaio  # search for info
		shell> yum install libaio # install library

5. 安装指令

		shell> groupadd mysql
		shell> useradd -r -g mysql -s /bin/false mysql
		shell> cd /usr/local
		shell> tar zxvf /path/to/mysql-VERSION-OS.tar.gz
		shell> ln -s full-path-to-mysql-VERSION-OS mysql
		shell> cd mysql
		shell> mkdir mysql-files
		shell> chown mysql:mysql mysql-files
		shell> chmod 750 mysql-files
		shell> bin/mysqld --initialize --user=mysql 
		shell> bin/mysql_ssl_rsa_setup              
		shell> bin/mysqld_safe --user=mysql &
		# Next command is optional
		shell> cp support-files/mysql.server /etc/init.d/mysql.server

6. 添加环境变量

	在`/etc/profile`文件末尾添加如下一行

		export PATH=$PATH:/usr/local/mysql/bin
	然后 `source /etc/profile`

7. others

	安装完成后可以看到`/usr/local/mysql`目录下的布局如下

		# root @ localhost in /usr/local/mysql [8:54:53] 
		$ pwd
		/usr/local/mysql
		
		# root @ localhost in /usr/local/mysql [8:54:54] 
		$ ls
		bin  COPYING  data  docs  include  lib  man  mysql-files  README  share  support-files
		
		# root @ localhost in /usr/local/mysql [8:54:55]

	其中data目录放的就是数据
## 事务 ##
## 日志 ##
## 基本概念 ##
### 分库分表 ###
关系型数据库本身比较容易成为系统瓶颈，单机存储容量、连接数、处理能力都有限。当单表的数据量达到1000W或100G以后，由于查询维度较多，即使添加从库、优化索引，做很多操作时性能仍下降严重。此时就要考虑对其进行切分了，切分的目的就在于减少数据库的负担，缩短查询时间。

数据切分根据其切分类型，可以分为两种方式：垂直（纵向）切分和水平（横向）切分。

#### 垂直切分 ####

垂直切分常见有**垂直分库**和**垂直分表**两种。

**垂直分库**就是根据业务耦合性，将关联度低的不同表存储在不同的数据库。做法与大系统拆分为多个小系统类似，按业务分类进行独立划分。与"微服务治理"的做法相似，每个微服务使用单独的一个数据库。如下图：

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/%E5%88%86%E5%BA%93%E5%88%86%E8%A1%A8-%E7%BA%B5%E5%90%91%E5%88%87%E5%88%86.png)


**垂直分表**是基于数据库中的"列"进行，某个表字段较多，可以新建一张扩展表，将不经常用或字段长度较大的字段拆分出去到扩展表中。在字段很多的情况下（例如一个大表有100多个字段），通过"大表拆小表"，更便于开发与维护，也能避免跨页问题。MySQL底层是通过数据页存储的，一条记录占用空间过大会导致跨页，造成额外的性能开销。另外数据库以行为单位将数据加载到内存中，这样表中字段长度较短且访问频率较高，内存能加载更多的数据，命中率更高，减少了磁盘IO，从而提升了数据库性能。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/%E5%9E%82%E7%9B%B4%E5%88%86%E8%A1%A8.png)

- 垂直切分的优点
	1. 解决业务系统层面的耦合，业务清晰
	2. 与微服务的治理类似，也能对不同业务的数据进行分级管理、维护、监控、扩展等
	3. 高并发场景下，垂直切分一定程度的提升IO、数据库连接数、单机硬件资源的瓶颈
- 垂直切分的缺点
	1. 部分表无法join，只能通过接口聚合方式解决，提升了开发的复杂度
	2. 分布式事务处理复杂
	3. 依然存在单表数据量过大的问题（需要水平切分）

#### 水平切分 ####

当一个应用难以再细粒度的垂直切分，或切分后数据量行数巨大，存在单库读写、存储性能瓶颈，这时候就需要进行水平切分了。

水平切分分为**库内分表**和**分库分表**，是根据表内数据内在的逻辑关系，将同一个表按不同的条件分散到多个数据库或多个表中，每个表中只包含一部分数据，从而使得单个表的数据量变小，达到分布式的效果。
![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/union-sharding.png)

**库内分表**只解决了单一表数据量过大的问题，但没有将表分布到不同机器的库上，因此对于减轻MySQL数据库的压力来说，帮助不是很大，大家还是竞争同一个物理机的CPU、内存、网络IO，最好通过**分库分表来**解决。比如腾讯云的DCDB就是**分库分表**的思路。

- 水平切分的优点：
	1. 不存在单库数据量过大、高并发的性能瓶颈，提升系统稳定性和负载能力
	2. 应用端改造较小，不需要拆分业务模块
- 水平切分的缺点：
	1. 跨分片的事务一致性难以保证
	2. 跨库的join关联查询性能较差
	3. 数据多次扩展难度和维护量极大

数据的垂直切分基本上能够简单的理解为依照表、模块来切分数据，而水平切分就不再是依照表或者是功能模块来切分了。一般来说，简单的水平切分主要是将某个访问极其频繁的表再依照某个字段的某种规则来分散到多个表之中。每一个表中包括一部分数据。

#### 分库分表带来的问题 ####

分库分表能有效地缓解单机和单库带来的性能瓶颈和压力，突破网络IO、硬件资源、连接数的瓶颈，同时也带来了一些问题。

1. **事务一致性问题**

	当更新内容同时分布在不同库中，不可避免会带来跨库事务问题。跨分片事务也是*分布式事务*，没有简单的方案，一般可使用"XA协议"和"两阶段提交"处理。

	分布式事务能最大限度保证了数据库操作的原子性。但在提交事务时需要协调多个节点，推后了提交事务的时间点，延长了事务的执行时间。导致事务在访问共享资源时发生冲突或死锁的概率增高。随着数据库节点的增多，这种趋势会越来越严重，从而成为系统在数据库层面上水平扩展的枷锁。
	
	*最终一致性*
	
	对于那些性能要求很高，但对一致性要求不高的系统，往往不苛求系统的实时一致性，只要在允许的时间段内达到最终一致性即可，可采用事务补偿的方式。与事务在执行中发生错误后立即回滚的方式不同，事务补偿是一种事后检查补救的措施，一些常见的实现方法有：对数据进行对账检查，基于日志进行对比，定期同标准数据来源进行同步等等。事务补偿还要结合业务系统来考虑。
2. **跨节点关联查询 join 问题**
	
	切分之前，系统中很多列表和详情页所需的数据可以通过sql join来完成。而切分之后，数据可能分布在不同的节点上，此时join带来的问题就比较麻烦了，考虑到性能，尽量避免使用join查询。

	解决这个问题的一些方法：
	
	- **全局表**：

		全局表，也可看做是"数据字典表"，就是系统中所有模块都可能依赖的一些表，为了避免跨库join查询，可以将这类表在每个数据库中都保存一份。这些数据通常很少会进行修改，所以也不担心一致性的问题。
	- **字段冗余**

		一种典型的反范式设计，利用空间换时间，为了性能而避免join查询。例如：订单表保存userId时候，也将userName冗余保存一份，这样查询订单详情时就不需要再去查询"买家user表"了。

		但这种方法适用场景也有限，比较适用于依赖字段比较少的情况。而冗余字段的数据一致性也较难保证，就像上面订单表的例子，买家修改了userName后，是否需要在历史订单中同步更新呢？这也要结合实际业务场景进行考虑。
	- **数据组装**

		在系统层面，分两次查询，第一次查询的结果集中找出关联数据id，然后根据id发起第二次请求得到关联数据。最后将获得到的数据进行字段拼装
	- **ER分片**

		关系型数据库中，如果可以先确定表之间的关联关系，并将那些存在关联关系的表记录存放在同一个分片上，那么就能较好的避免跨分片join问题。如下图所示：
		
		![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/er%E5%88%86%E7%89%87.png)


		这样一来，Data Node1上面的order订单表与orderdetail订单详情表就可以通过orderId进行局部的关联查询了，Data Node2上也一样。

3. **跨节点分页、排序、函数问题**

	跨节点多库进行查询时，会出现limit分页、order by排序等问题。分页需要按照指定字段进行排序，当排序字段就是分片字段时，通过分片规则就比较容易定位到指定的分片。

	当排序字段非分片字段时，就变得比较复杂了。需要先在不同的分片节点中将数据进行排序并返回，然后将不同分片返回的结果集进行汇总和再次排序，最终返回给用户。如下图所示：

	![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/%E5%88%86%E8%A1%A8%E5%90%8Eselect.png)

	
	上图中只是取第一页的数据，对性能影响还不是很大。但是如果取得页数很大，情况则变得复杂很多，因为各分片节点中的数据可能是随机的，为了排序的准确性，需要将所有节点的前N页数据都排序好做合并，最后再进行整体的排序，这样的操作是很耗费CPU和内存资源的，所以页数越大，系统的性能也会越差。

	在使用Max、Min、Sum、Count之类的函数进行计算的时候，也需要先在每个分片上执行相应的函数，然后将各个分片的结果集进行汇总、再次计算，最终将结果返回。

4. **全局主键避重问题**
	
	在分库分表环境中，由于表中数据同时存在不同数据库中，主键平时使用的自增长将无用武之地，某个分区数据库自生成的ID无法保证全局唯一。**因此需要单独设计全局主键**，以避免跨库主键重复问题。有一些常见的主键生成策略。

	- UUID

		UUID标准形式包含32个16进制数字，分为5段，形式为8-4-4-4-12的36个字符，例如：550e8400-e29b-41d4-a716-446655440000
		
		UUID是主键是最简单的方案，本地生成，性能高，没有网络耗时。但缺点也很明显，由于UUID非常长，会占用大量的存储空间；另外，作为主键建立索引和基于索引进行查询时都会存在性能问题，**在InnoDB下，UUID的无序性会引起数据位置频繁变动**，导致分页。
	- 结合数据库维护主键ID表
		
		在数据库中建立 sequence 表：

			CREATE TABLE `sequence` (  
	  			`id` bigint(20) unsigned NOT NULL auto_increment,  
	  			`stub` char(1) NOT NULL default '',  
	  			PRIMARY KEY  (`id`),  
	  			UNIQUE KEY `stub` (`stub`)  
			) ENGINE=MyISAM;

		stub字段设置为唯一索引，同一stub值在sequence表中只有一条记录，可以同时为多张表生成全局ID。sequence表的内容，如下所示：

			+-------------------+------+  
			| id                | stub |  
			+-------------------+------+  
			| 72157623227190423 |    a |  
			+-------------------+------+  

		使用 MyISAM 存储引擎而不是 InnoDB，以获取更高的性能。MyISAM使用的是表级别的锁，对表的读写是串行的，所以不用担心在并发时两次读取同一个ID值。

		当需要全局唯一的64位ID时，执行：

			REPLACE INTO sequence (stub) VALUES ('a');  
			SELECT LAST_INSERT_ID();  

		这两条语句是Connection级别的，select last_insert_id() 必须与 replace into 在同一数据库连接下才能得到刚刚插入的新ID。

		使用replace into代替insert into好处是避免了表行数过大，不需要另外定期清理。

		此方案较为简单，但缺点也明显：**存在单点问题，强依赖DB，当DB异常时，整个系统都不可用。配置主从可以增加可用性，但当主库挂了，主从切换时，数据一致性在特殊情况下难以保证。另外性能瓶颈限制在单台MySQL的读写性能**。

		为了解决单点和性能瓶颈问题，flickr提出如下方案。

		![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/flickr_uniqueId.png)

		建立2个以上的全局ID生成的服务器，每个服务器上只部署一个数据库，每个库有一张sequence表用于记录当前全局ID。表中ID增长的步长是库的数量，起始值依次错开，这样能将ID的生成散列到各个数据库上。

		由两个数据库服务器生成ID，设置不同的auto_increment值。第一台sequence的起始值为1，每次步长增长2；另一台的sequence起始值为2，每次步长增长也是2。结果第一台生成的ID都是奇数（1, 3, 5, 7 ...），第二台生成的ID都是偶数（2, 4, 6, 8 ...）。

		这种方案将生成ID的压力均匀分布在两台机器上。同时提供了系统容错，第一台出现了错误，可以自动切换到第二台机器上获取ID。但有以下几个缺点：系统添加机器，水平扩展时较复杂；每次获取ID都要读写一次DB，DB的压力还是很大，只能靠堆机器来提升性能。

		可以基于flickr的方案继续优化，使用批量的方式降低数据库的写压力，每次获取一段区间的ID号段，用完之后再去数据库获取，可以大大减轻数据库的压力。如下图所示：

		![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/max_gen.png)

		还是使用两台DB保证可用性，数据库中只存储当前的最大ID。ID生成服务每次批量拉取6个ID，先将max_id修改为5，当应用访问ID生成服务时，就不需要访问数据库，从号段缓存中依次派发0~5的ID。当这些ID发完后，再将max_id修改为11，下次就能派发6~11的ID。于是，数据库的压力降低为原来的1/6。


	- 订单
	
	参考：[Leaf-美团点评分布式ID生成系统](https://tech.meituan.com/MT_Leaf.html)