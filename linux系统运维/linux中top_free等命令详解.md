# 一些查看当前机器资源使用情况的linux命令 #
## top命令 ##

在linux机器上执行`top`命令可以看到cpu利用率的数据。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/top_cmd.png)


前五行是系统整体的统计信息。第一行是任务队列信息，同 uptime 命令的执行结果。其内容如下：

- 17：10：04

  表示当前时间

- up 255 days, 6:24

  系统运行时间，表示运行了255天6小时24分

- 13 users

  当前登录用户数

- load average:0.06,0.08.0.14

  系统负载

- Tasks: 206 total

  进程总数

- 1 running

  正在运行的进程数目

- 205 sleeping

  睡眠的进程数（A sleeping process may be waiting on something -- input/output, a child process to return, etc.）。

  关于sleeping process，引用一段话解释：


  > Suppose a process starts: it gets loaded into memory, so some memory has to be allocated to it. It also gets some processor time, otherwise it would lurk just there, unable to run. Now it starts and probably it will need some additional memory to hold runtime data, it might need other OS resources, like files to be opened, network connections to be established, etc., etc..


  > All these requests involve the OS, which may or may not be able to fulfill these immediately. If it is, the process gets what it requests, but if not it will be put to sleep until the OS can provide. That it is put to sleep does not mean it has nothing to do or that it could be stopped. This is just a way for the OS to do something else until it can provide everything necessary to run the process.


  > Another possiblity is that the process waits for a certain event: suppose the process services a certain network event: it will listen to the network and until a certain signal comes it has nothing to do - therefore it is going to sleep. When the signal comes, it wakes up, does whatever it is supposed to do, then goes back to sleep again. If you stop the process because it sleeps it will not be able to wake up once the signal comes.

- 0 stopped

  停止的进程数

- 0 zombie

  僵尸进程数

- Cpu(s):0.6%us,...,0.0%st

  关于us，sy，……，参见后面的解释

- Mem: 8046144k total,...,369348k buffers
  1. 8046144k total

     物理内存的总量

  2. 4442124k used

     使用的物理内存总量

  3. 3604020k free

     空闲内存总量

  4. 369348k buffers

     用作内核缓存的内存量

- Swap: 2097148k,...,2438076k cached
  1. 2097148k total

     交换区总量

  2. 194804k used

     使用的交换区总量

  3. 1902344k free

     空闲交换区总量

  4. 2438076k cached

     缓冲的交换区总量

top 命令后，按键盘`1`，可以得到各个cpu的使用情况，如下：

	top - 19:58:17 up 252 days,  9:12, 12 users,  load average: 0.12, 0.22, 0.18
	Tasks: 206 total,   1 running, 205 sleeping,   0 stopped,   0 zombie
	Cpu0  :  1.3%us,  4.3%sy,  0.0%ni, 94.4%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
	Cpu1  :  1.3%us,  0.3%sy,  0.0%ni, 98.3%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
	Cpu2  :  1.0%us,  0.0%sy,  0.0%ni, 99.0%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
	Cpu3  :  0.7%us,  0.3%sy,  0.0%ni, 99.0%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
	Mem:   8046144k total,  6801424k used,  1244720k free,   182348k buffers
	Swap:  2097148k total,    80336k used,  2016812k free,  4881488k cached
	
	  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND                                                                             
	30521 root      20   0 45768  23m 5480 S  1.7  0.3   1069:46 sap1009                                                                             
	25502 root      20   0  791m  40m  10m S  1.3  0.5   1:43.72 node                                                                                
	19486 root      20   0  888m  68m  12m S  1.0  0.9  31:04.22 squirrel-worker                                                                     
	30506 root      20   0  7180 6132  668 S  1.0  0.1   2284:18 sap1002

这里涉及到`us`,`sy`,`ni`,`id`,`wa`,`hi`,`si`和`st`这些指标，分别是什么含义呢？

引用一下top命令的man-pages的介绍。

> us, user: time running un-niced user processes
>
> sy, system: time running kernel processes
>
> ni, nice: time running niced user processes
>
> id, idle: time spent in the kernel idle handler
>
> wa, IO-wait: time waiting for I/O completion
>
> hi: time spent servicing hardware interrupts
>
> si: time spent servicing software interrupts
>
> st: time stolen from this vm by the hypervisor

中文解释就是：

- us

  用户空间占用CPU百分比（仅包括未改变优先级的）
- sy

  内核空间占用CPU百分比
- ni

  用户进程空间内改变过优先级的进程占用CPU百分比
- id

  空闲CPU百分比
- wa

  IO等待所占用的CPU时间的百分比
- hi

  CPU服务于硬中断所耗费的时间总额
- si

  CPU服务于软中断所耗费的时间总额

- st

  hypervisor（一种运行在物理服务器和操作系统之间的中间软件层，可允许多个操作系统和应用共享一套基础物理硬件）从当前虚拟机偷走的cpu时间。这个指标是针对虚拟机的。

  Steal 值比较高的话，你需要向主机供应商申请扩容虚拟机。服务器上的另一个虚拟机可能拥有更大更多的 CPU 时间片，你可能需要申请升级以与之竞争。另外，高 steal 值可能意味着主机供应商在服务器上过量地出售虚拟机。如果升级了虚拟机， steal 值还是不降的话，你应该寻找另一家服务供应商。


## free 命令 ##

free 命令用于查看当前系统的内存使用情况。

	[root@VM_126_59_tlinux /data/tdsql_shark/log]# free -m
	             total       used       free     shared    buffers     cached
	Mem:          7857       4654       3203          0        382       2665
	-/+ buffers/cache:       1606       6251
	Swap:         2047        188       1859
	[root@VM_126_59_tlinux /data/tdsql_shark/log]# 

对于上面的例子，解释如下：

- Mem_total

  内存总数，7857

- Mem_used

  已经使用的内存数（包含buffers 与cache），4654

- Mem_free

  空闲的内存数，3203

- Mem_shared

  共享内存，一般系统不会用到，这里也不讨论

- Mem_buffers
- Mem_cached
- -buffers/cache

  1606，等于Mem_used-Mem_buffers-Mem_cached。这里可能有出入，算了一下，Mem_used-Mem_buffers-Mem_cached=4654-382-2665=1607,而Mem_used为1606。

  反映的是被程序实实在在吃掉的内存。

- +buffers/cache

  6251，等于Mem_free+Mem_buffers+Mem_cached。这里也有出入，算了一下，Mem_free+Mem_buffers+Mem_cached=3203+382+2665=6250,而从上面`free -m`命令执行的结果来看，`+buffers/cache`为6251。

  反映的是可以挪用的内存总数

- Swap_total
- Swap_used
- Swap_free

### buffer和cache的区别 ###
>
> A buffer is something that has yet to be "written" to disk. A cache is something that has been "read" from the disk and stored for later use.

两者都是RAM中的数据。简单来说，buffer是即将要被写入磁盘的，cache是被从磁盘中读出来的。

buffer是由各种进程分配的，被用在如输入队列等方面，一个简单的例子如某个进程要求有多个字段读入，在所有字段被读入完整之前，进程把先前读入的字段放在buffer中保存。

cache经常被用在磁盘的I/O请求上，如果有多个进程都要访问某个文件，于是该文件便被做成cache以方便下次被访问，这样可提供系统性能。

## linux计算cpu使用率 ##

计算cpu使用率可以使用`/proc/stat`。

`/proc/stat`存储的是系统的一些统计信息。 该文件包含了所有CPU活动的信息，该文件中的所有值都是从系统启动开始累计到当前时刻。

我机器上执行得到的结果是：

	# victordong @ VM_144_188_centos in /data/victor/10_2_7/shark/test [20:24:41] 
	$ cat /proc/stat 
	cpu  19185968 82 6873228 936299176 206814 42 131492 0 0 0
	cpu0 5309894 20 1758535 233441064 66049 3 6333 0 0 0
	cpu1 5227752 30 1769806 233554369 35049 10 43194 0 0 0
	cpu2 4301302 14 1701117 234622574 70243 20 34429 0 0 0
	cpu3 4347019 16 1643768 234681168 35472 8 47536 0 0 0
	intr 3841658299 34 10 0 0 516 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 3147852 0 2847094 0 69 0 48242894 58 52661929 1633 72376645 1370 70916602 1319 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
	ctxt 7617637925
	btime 1539175483
	processes 13403606
	procs_running 1
	procs_blocked 0
	softirq 1372221024 0 282927161 0 670824493 0 0 1 181166778 255847 237046744
	
	# victordong @ VM_144_188_centos in /data/victor/10_2_7/shark/test [20:24:49] 
	$ 

上面数值的所有单位都是jiffies，jiffies是内核中的一个全局变量，用来记录自系统启动一来产生的节拍数，在linux中，一个节拍大致可理解为操作系统进程调度的最小时间片，不同linux内核可能值有不同，通常在1ms到10ms之间。

下面解释一下各数值的含义。



