# 一些查看当前机器资源使用情况的linux命令 #
## top命令 ##

在linux机器上执行`top`命令可以看到cpu利用率的数据。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/top_cmd.png)


前五行是系统整体的统计信息。第一行是任务队列信息，同 `uptime` 命令的执行结果。其内容如下：

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

  	停止的进程数。进程收到SIGSTOP信号后进入该状态，在收到SIGCONT后又会恢复运行。
  	
  	比如当一个进程可以和用户进行交互时，如果你通过`&`符号，将其放入到后台运行，那么这个进程就是stopped状态，比如`top &`。


- 0 zombie

  	僵尸进程数。

	一个进程在调用exit命令结束自己生命的时候，其实它并没有真正地被销毁，而是留下一个称为僵尸进程（Zombie）的数据结构（系统调用exit， 它的作用是使进程退出，但也仅仅限于将一个正常的进程变成一个僵尸进程，并不能将其完全销毁）。

	在Linux进程的状态中，僵尸进程是非常特殊的一种，它已经放弃了几乎所有内存空间，没有任何可执行代码，也不能被调度，仅仅在进程列表中保留一个位置，记录该进程的退出状态等信息供其他进程收集，除此之外，僵尸进程不再占有任何内存空间。

	它需要它的父进程来为它收尸，如果他的父进程没安装SIGCHLD信号处理函数，调用wait或waitpid()等待子进程结束，又没有显式忽略该信号，那么它就一直保持僵尸状态，如果这时父进程结束了，那么init进程会自动接手这个子进程，为它收尸，它还是能被清除的。

	但是如果父进程是一个循环，不会结束，那么子进程就会一直保持僵尸状态，这就是为什么系统中有时会有很多的僵尸进程。

- Cpu(s):0.6%us,...,0.0%st

  	关于us，sy，……，参见后面的解释

- Mem: 8046144k total,...,369348k buffers
  1. 8046144k total

     	物理内存的总量

  2. 4442124k used

     	使用的物理内存总量

  3. 3604020k free

     	空闲内存总量。used + free = total。

  4. 369348k buffers

     	用作内核缓存的内存量

- Swap: 2097148k,...,2438076k cached
  1. 2097148k total

     	交换区总量

  2. 194804k used

     	使用的交换区总量

  3. 1902344k free

     	空闲交换区总量。used + free = total。

  4. 2438076k cached

    	缓冲的交换区总量

再来看上面截图中，第六行后面的部分。这部分是各个进程占用的资源情况。

- PID		进程号
- USER		进程创建者
- PR		进程优先级
- NI		nice值。越小优先级越高，最小-20，最大19

	>The nice value of the task.  A negative nice value means higher priority, whereas a positive nice value means lower priority.  Zero in this field simply means priority  will
- VIRT		进程使用的虚拟内存总量
- RES		进程使用的，未被换出的物理内存大小
- SHR		共享内存大小
- S			进程状态
- %CPU		进程占用CPU百分比
- %MEM		进程占用内存百分比
- TIME+		进程运行时间
- COMMAND	进程名称

上面有几个概念可能比较难理解，解释下。



**NI和PR**

也就是nice 和 priority。


nice 取值范围为[-20, 19]，值越小，优先级越高，静态优先级。

在内核中，进程优先级的取值范围是通过一个宏定义的，这个宏名称是MAX_PRIO，取值为140。

而这个值又是由另外两个值相加组成的，一个是代表nice值取值范围的`NICE_WIDTH`宏（40），另一个是代表实时进程（realtime）优先级范围的`MAX_RT_PRIO`宏（100）。

说白了就是，Linux实际上实现了140个优先级，取值范围是从0-139，这个值越小，优先级越高。

nice值的-20到19，映射到实际的优先级范围是100-139，这是针对正常的进程来说的（非实时进程）。

	PR = 20 + NI

也就是说，如果nice为0的话，呢么PR就是20（你可以通过top看到，大部分的进程PR都是20），如果nice为-20的话，PR就是0，对于正常的进程来说，PR为0就是优先级最高的了。

这里也可以看到，PR为0的话，对应的就是linux内核中140个优先级[0, 139]的100。

PR = 20 + (-20 to +19) 就是0到39，映射为100到139。优先级100以下的都是rt（real time）进程。


如果对于real time 进程，那么

	PR = -1 - rt_prior

其中rt_prior取值为[1, 99]。

当`rt_prior`为99的时候，PR为-100，对应linux内核中140个优先级[0, 139]的0，优先级最高。

**VIRT**

virtual memory usage。虚拟内存使用？

**包括进程使用的库、代码、数据，以及malloc、new分配的堆空间和分配的栈空间等**。

假如进程新申请10MB的内存，但实际只使用了1MB，那么它会增长10MB，而不是实际的1MB使用量。

我们知道每个进程面对的是一个独立的地址空间，比如进程A面对的地址空间是0x0000~0xFFFF，进程B面对的地址空间也是0x0000~0xFFFF。

但是我们内存条只有一块，那进程A和进程B的地址重复了怎么办？

其实操作系统给我们做了一个映射，可以将进程的虚拟地址映射到内存条的实际地址中去。这样即使进程A和进程B的虚拟地址是一样的，但是实际上在内存条的物理地址是不一样的。具体相关知识可以参考《深入理解计算机操作系统一书》。

那这里VIRT表示的其实是进程占有的一个地址空间，只要是应用程序要求的，就全算在这里，而不管它真的用了没有，也不管有没有实际映射到物理内存中去。

比如你在堆里new了一个大数组，类似这样，`char *p = new char [1024*1024*256]`，那么程序占用的VIRT大概就会是256M。但是RES却很可能没那么大。

	top - 20:00:17 up 13 days, 18:47,  1 user,  load average: 0.15, 0.15, 0.14
	Tasks:   1 total,   0 running,   1 sleeping,   0 stopped,   0 zombie
	%Cpu(s):  0.3 us,  0.3 sy,  0.0 ni, 98.7 id,  0.7 wa,  0.0 hi,  0.0 si,  0.0 st
	%Node0 :  0.3 us,  0.3 sy,  0.0 ni, 98.7 id,  0.7 wa,  0.0 hi,  0.0 si,  0.0 st
	KiB Mem :  1015564 total,   133224 free,   559352 used,   322988 buff/cache
	KiB Swap:        0 total,        0 free,        0 used.   298484 avail Mem 
	
	  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                                                                  
	22241 root      20   0  274684   1068    900 S  0.0  0.1   0:00.00 a.out           

测试代码是：


	#include <iostream>
	#include <stdio.h>
	int main()
	{
	        char *p = new char[1024*1024*256];
	        getchar();
	        return 0;
	}

可以看到，我们new了一个256M（268435456字节）大小的数组。top命令查看得到的占用VIRT的空间是274684KiB（281276416字节），


![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/virt.png)

但是实际的res占用并不高。

**RES**

进程当前使用的内存大小，包括**使用中**的malloc、new分配的堆空间和分配的栈空间，但不包括swap out量。

包含其他进程的共享。

如果申请10MB的内存，实际使用1MB，它只增长1MB，与VIRT相反。

关于库占用内存的情况，它只统计加载的库文件所占内存大小。



**SHR**


>The amount of shared memory used by a task. It simply reflects memory that could be potentially shared with other processes. 

共享内存。

除了自身进程的共享内存，也包括其他进程的共享内存。

虽然进程只使用了几个共享库的函数，但它包含了整个共享库的大小。

计算某个进程所占的物理内存大小公式：RES – SHR。


### 查看各个cpu的使用情况 ###

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

	IO等待所占用的CPU时间的百分比（io等待，cpu不应该是空闲状态么？）
- hi

	CPU服务于硬中断所耗费的时间总额
- si

	CPU服务于软中断所耗费的时间总额

- st

	hypervisor（一种运行在物理服务器和操作系统之间的中间软件层，可允许多个操作系统和应用共享一套基础物理硬件）从当前虚拟机偷走的cpu时间。这个指标是针对虚拟机的。

	Steal 值比较高的话，你需要向主机供应商申请扩容虚拟机。服务器上的另一个虚拟机可能拥有更大更多的 CPU 时间片，你可能需要申请升级以与之竞争。另外，高 steal 值可能意味着主机供应商在服务器上过量地出售虚拟机。如果升级了虚拟机， steal 值还是不降的话，你应该寻找另一家服务供应商。


### top常用操作 ###

1. 按照cpu使用率对进程进行排序

	进入top界面后，按`shift + p`，或者大写锁定后，按P。
	
2. 设置top 的刷新间隔

	`top -d 1`，按照一秒间隔刷新。或者进入top界面后，按d更改刷新频率。
3. 查看指定进程的情况

		`top -p`
4. 按照内存使用率进行排序

	进入top界面后，`shift + m`或者大写锁定后按M。按照实际使用的物理内存排序的，也就是res。
	
5. 显示每个cpu的情况

	进入top界面后，按数字1。
	
6. 指定用户进程

	进入top界面后，按u。
	
7. 按照pid排序

	进入top界面后，按`shift + n`，或者大写锁定按N。

8. 在多个不同列进行切换要排序的展示列。

	进入top界面后，`shift + <`或者`shift + >`。左移或者右移排序列。一般会和该表调色板（按z）一起配合使用。
	
	进入top界面后，按x，以field进行展示。
	
	我一般也会按b，调整下是黑体还是反色显示排序的列。效果类似下面这样，
	
	
	![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/top_sort.png)
		
	
9. 反向排序

		shift + r	

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

buffer是由各种进程分配的，被用在如输入队列等方面。一个简单的例子，如某个进程要求有多个字段被读入，在所有字段被读入完整之前，进程把先前读入的字段放在buffer中保存。

cache经常被用在磁盘的I/O请求上，如果有多个进程都要访问某个文件，于是该文件便被做成cache以方便下次被访问，这样可提高系统性能。


Buffer（缓冲区）是系统两端处理速度平衡（从长时间尺度上看）时使用的。它的引入是为了减小短期内突发I/O的影响，起到流量整形的作用。比如生产者——消费者问题，他们产生和消耗资源的速度大体接近，加一个buffer可以抵消掉资源刚产生/消耗时的突然变化。

Cache（缓存）则是系统两端处理速度不匹配时的一种折衷策略。因为CPU和memory之间的速度差异越来越大，所以人们充分利用数据的局部性（locality）特征，通过使用存储系统分级（memory hierarchy）的策略来减小这种差异带来的影响。
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

- user(19185968)，从系统启动开始累计到当前时刻，用户态的CPU时间（单位：jiffies） 。
- nice(82)，从系统启动开始累计到当前时刻，低优先级程序所占用的用户态的CPU时间（单位：jiffies）。
- system(6873228)，从系统启动开始累计到当前时刻，内核态时间。
- idle(936299176)，从系统启动开始累计到当前时刻，除硬盘IO等待时间以外其它空闲时间。

	这里说**除硬盘IO等待时间以外其它空闲时间**，是因为有可能在等io的时候，cpu被调度去干其他事情了
- iowait(206814)，从系统启动开始累计到当前时刻，等待io响应的时间。

	这里等待io响应的时间是这样定义的：**CPU空闲、并且有仍未完成的I/O请求**。
	
	**大白话就是cpu闲着啥事不干等io完成的时间**。
	
	假如说cpu很忙，进程A在做io，cpu很可能就被调度走去干其他事情了。那进程A做io的这段时间，是不会算进到iowait里面去的。
	
	再比如，在CPU繁忙期间发生的I/O，无论IO是多还是少，%iowait都不会变；当CPU繁忙程度下降时，有一部分IO落入CPU空闲时间段内，导致%iowait升高。
	
	IO的并发度低，%iowait就高；IO的并发度高，%iowait可能就比较低。
	
	我们可以这样认为，当系统的cpu占用率不高，cpu很空闲时，如果%iowait升高，有进程在做大io操作，cpu都在等io。
	
	可见%iowait是一个非常模糊的指标，如果看到 %iowait 升高，还需检查I/O量有没有明显增加，avserv/avwait/avque等指标有没有明显增大，应用有没有感觉变慢，如果都没有，就没什么好担心的。
	
	我们验证下：
	
		[root@victor2 ~]# dd if=/dev/zero of=test.dbf bs=8k count=3000000
	
	先利用dd命令做大io操作。top查看iowait确实很高。
	
	然后我们再执行一个死循环进程`a.out`。
		[root@victor2 ~]# ./a.out 
		
	再看一下top，发现iowait降低几乎为0了。
	
- irq(42)，从系统启动开始累计到当前时刻，处理硬中断的时间。
- softirq(131492)，从系统启动开始累计到当前时刻，处理软中断的时间。
- steal(0)，在虚拟环境下 CPU 花在处理其他作业系统的时间，Linux 2.6.11 开始才开始支持。
- guest(0)，在 Linux 内核控制下 CPU 为 guest 作业系统运行虚拟 CPU 的时间，Linux 2.6.24 开始才开始支持。
- guest_nice(0)（since Linux 2.6.33），Time spent running a niced guest (virtual CPU for guest operating systems under the control of the Linux kernel).

**网络上很多关于nice的解释是错的，将其解释为，nice值为负的进程所占用的CPU时间。**

**对user的解释也有问题，	解释为用户态的CPU时间（单位：jiffies），不包含nice值为负进程。**

其实，我们看下官方的文档就知道了（出处：[http://man7.org/linux/man-pages/man5/proc.5.html](http://man7.org/linux/man-pages/man5/proc.5.html)）

	/proc/stat
      kernel/system statistics.  Varies with architecture.  Common
      entries include:

      cpu 10132153 290696 3084719 46828483 16683 0 25195 0 175628 0
      cpu0 1393280 32966 572056 13343292 6130 0 17875 0 23933 0
             The amount of time, measured in units of USER_HZ
             (1/100ths of a second on most architectures, use
             sysconf(_SC_CLK_TCK) to obtain the right value), that
             the system ("cpu" line) or the specific CPU ("cpuN"
             line) spent in various states:

             user   (1) Time spent in user mode.

             nice   (2) Time spent in user mode with low priority
                    (nice).

             system (3) Time spent in system mode.

             idle   (4) Time spent in the idle task.  This value
                    should be USER_HZ times the second entry in the
                    /proc/uptime pseudo-file.

             iowait (since Linux 2.5.41)
                    (5) Time waiting for I/O to complete.  This
                    value is not reliable, for the following rea‐
                    sons:

                    1. The CPU will not wait for I/O to complete;
                       iowait is the time that a task is waiting for
                       I/O to complete.  When a CPU goes into idle
                       state for outstanding task I/O, another task
                       will be scheduled on this CPU.

                    2. On a multi-core CPU, the task waiting for I/O
                       to complete is not running on any CPU, so the
                       iowait of each CPU is difficult to calculate.

                    3. The value in this field may decrease in cer‐
                       tain conditions.

             irq (since Linux 2.6.0)
                    (6) Time servicing interrupts.

             softirq (since Linux 2.6.0
                    (7) Time servicing softirqs.

             steal (since Linux 2.6.11)
                    (8) Stolen time, which is the time spent in
                    other operating systems when running in a virtu‐
                    alized environment

             guest (since Linux 2.6.24)
                    (9) Time spent running a virtual CPU for guest
                    operating systems under the control of the Linux
                    kernel.

             guest_nice (since Linux 2.6.33)
                    (10) Time spent running a niced guest (virtual
                    CPU for guest operating systems under the con‐
                    trol of the Linux kernel).

      page 5741 1808
             The number of pages the system paged in and the number
             that were paged out (from disk).

      swap 1 0
             The number of swap pages that have been brought in and
             out.

      intr 1462898
             This line shows counts of interrupts serviced since
             boot time, for each of the possible system interrupts.
             The first column is the total of all interrupts ser‐
             viced including unnumbered architecture specific inter‐
             rupts; each subsequent column is the total for that
             particular numbered interrupt.  Unnumbered interrupts
             are not shown, only summed into the total.

      disk_io: (2,0):(31,30,5764,1,2) (3,0):...
             (major,disk_idx):(noinfo, read_io_ops, blks_read,
             write_io_ops, blks_written)
             (Linux 2.4 only)

      ctxt 115315
             The number of context switches that the system under‐
             went.

      btime 769041601
             boot time, in seconds since the Epoch, 1970-01-01
             00:00:00 +0000 (UTC).

      processes 86031
             Number of forks since boot.

      procs_running 6
             Number of processes in runnable state.  (Linux 2.5.45
             onward.)

      procs_blocked 2
             Number of processes blocked waiting for I/O to com‐
             plete.  (Linux 2.5.45 onward.)

      softirq 229245889 94 60001584 13619 5175704 2471304 28 51212741 59130143 0 51240672
             This line shows the number of softirq for all CPUs.
             The first column is the total of all softirqs and each
             subsequent column is the total for particular softirq.
             (Linux 2.6.31 onward.)


上面文档可以看出，`nice`的含义是低优先级的process所花的时间，低优先级其实应该是nice为正的进程。

`user`的含义是不包括低优先级的process运行时间的用户进程的时间。

我们也可以验证下。

	[root@victor2 ~]# cat test.cpp 
	#include <iostream>
	#include <stdio.h>
	int main()
	{
	        while(1);
	        return 0;
	}
	[root@victor2 ~]# 
	[root@victor2 ~]# nice -n 1 ./a.out

`a.out`是`test.cpp`编译产物，`nice -n 1 ./a.out`表示调整`a.out`运行nice值为1。

通过top看cpu占用率，如下，
![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/nice_1.png)

可以发现ni为99.7。

如果我们这样呢，

	[root@victor2 ~]# nice -n -1 ./a.out   
	
top查看cpu利用率，

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/nice_2.png)

可以发现ni还是为0，但是us是99.7。

验证得到的结果就是。

1. ni表示的是运行时，通过nice命令调整了nice值>0的进程所用的时间。

	>The ni stat shows how much time the CPU spent running user space processes that have been niced. On a system where no processes have been niced then the number will be 0.
2. us表示的是用户空间进程所花时间（不包括1中所花时间）


github上一个简单的计算cpu使用率的shell脚本，[这里](https://github.com/pcolby/scripts/blob/master/cpu.sh)。

也有使用c代码实现计算cpu使用率的，[这里](https://github.com/scaidermern/top-processes/blob/master/top_proc.c)。

### 计算某个进程的cpu使用率 ###

我们可以通过读取`/proce/[pid]/stat`文件里面的内容，来计算某个进程的cpu使用率。

	[root@victor2 ~]# cat /proc/18069/stat
	18069 (a.out) R 17945 18069 17945 34816 18069 4202496 338 0 0 0 41112 7 0 0 20 0 1 0 164339492 12832768 206 18
	446744073709551615 4194304 4196508 140730399340208 140730399335608 4195889 0 0 0 0 0 0 0 17 0 0 0 0 0 0 629500
	0 6295612 33730560 140730399348743 140730399348751 140730399348751 140730399350768 0
	[root@victor2 ~]# 

- pid  %d (18069)
- comm  %s (a.out)
- state  %c (R)

	进程状态。R表示running状态。
- ppid  %d (17945)
- pgrp  %d (18069)

	进程组。

	The process group ID of the process.
	
- session  %d (17945)

	The session ID of the process.
	
	
- tty_nr  %d (34816)
- tpgid  %d
- flags  %u

- minflt  %lu (18069)

- cminflt  %lu (4202496)

- majflt  %lu (338)

- cmajflt  %lu (0)
- utime %lu

	>Amount of time that this process has been scheduled in user mode, measured in clock ticks (divide by sysconf(_SC_CLK_TCK)).  This includes guest time, guest\_time (time spent running a virtual CPU, see below), so that applications that are not aware of the guest time field do not lose that time from their calculations.
	
	该任务在用户态运行的时间，单位为jiffies。
	
- stime  %lu
	>Amount of time that this process has been scheduled in kernel mode, measured in clock ticks (divide by sysconf(_SC_CLK_TCK)).
	
	该任务在核心态运行的时间，单位为jiffies
	
	
- cutime  %ld
- cstime  %ld
- priority  %ld
- nice  %ld
- num_threads  %ld
- itrealvalue  %ld
- starttime  %llu
- vsize  %lu


## linxu 计算io使用率 ##


采集指标，[这里](http://km.oa.com/group/568/articles/show/197149)。