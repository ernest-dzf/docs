# 查看机器cpu信息 #

1. 查看物理cpu个数，这里的cpu个数是指插在主板上的cpu个数。
	```
  [user_00@TENCENT64 ~]$ cat /proc/cpuinfo | grep "physical id" | sort | uniq | wc -l
  2
  [user_00@TENCENT64 ~]$ 
	```
2. 查看一个cpu的物理核个数，我们常说的双核cpu，四核cpu就是说的这个。
	```
  [user_00@TENCENT64 ~]$ cat /proc/cpuinfo | grep "cpu cores" | uniq
  cpu cores       : 6
  [user_00@TENCENT64 ~]$ 
	```
3. 查看系统CPU是否启用超线程
	```
  [user_00@TENCENT64 ~]$ cat /proc/cpuinfo | grep -e "cpu cores"  -e "siblings" | sort | uniq
  cpu cores       : 6
  siblings        : 12
  [user_00@TENCENT64 ~]$ 
	```
	如果cpu cores数量和siblings数量一致，则没有启用超线程，否则超线程被启用。上面就是启用了超线程的。
	
4. 查看系统的逻辑核个数
	```
  [user_00@TENCENT64 ~]$ cat /proc/cpuinfo | grep "processor" | wc -l
  24
  [user_00@TENCENT64 ~]$ 
	```

以上面1、2、3、4例子来看，这个机器有2个物理CPU，每个CPU有6个物理核，每个CPU开启了超线程，有12个逻辑核。该机器总共有24个逻辑核，对应的是`cat /proc/cpuinfo`输出信息中的`processor0`,`processor1`,`processor2`,……。
		
# 跑cpu占用率 #

测试脚本如下：

	#! /bin/sh  
	# filename killcpu.sh 
	if [ $# != 1 ] ; then
	  echo "USAGE: $0 <CPUs>"
	  exit 1;
	fi
	for i in `seq $1`
	do
	  echo -ne "  
	i=0;  
	while true 
	do 
	i=i+1;  
	done" | /bin/sh &
	  pid_array[$i]=$! ;
	done
	
	for i in "${pid_array[@]}"; do
	  echo 'kill ' $i ';';
	done


使用方法如下：

	[root@TENCENT64 /home/user_00]# ls
	backup  install  killcpu.sh  services  test
	[root@TENCENT64 /home/user_00]# sh killcpu.sh 3
	kill  24489 ;
	kill  24491 ;
	kill  24493 ;
	[root@TENCENT64 /home/user_00]# 
