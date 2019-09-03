

# zookeeper

## 准备三台机器

这里使用3台虚拟机，分别配置免密登录。

- vm1，172.16.77.128
- vm2，172.16.77.138
- vm3，172.16.77.139



修改一下各自的hostname，方便查看。

centos7的话，使用`hostnamectl set-hostname vm3`这种。

配置下各自的`/etc/hosts`文件，可以通过hostname访问。

## JVM安装

1. [下载j](https://www.oracle.com/technetwork/java/jdk8-downloads-2133151.html)dk，解压

   ```shell
   [root@vm3 jdk1.8.0_221]# pwd
   /root/jdk/jdk1.8.0_221
   [root@vm3 jdk1.8.0_221]# ls
   bin        javafx-src.zip  LICENSE      release                             THIRDPARTYLICENSEREADME.txt
   COPYRIGHT  jre             man          src.zip
   include    lib             README.html  THIRDPARTYLICENSEREADME-JAVAFX.txt
   [root@vm3 jdk1.8.0_221]# 
   
   	
   ```

   

2. 安装jdk，配置下环境变量。配置`/etc/profile`文件，添加如下

   ```shell
   JAVA_HOME=/root/jdk/jdk1.8.0_221                     
   export PATH=$JAVA_HOME/bin:$PATH	
   ```

3. source一些`/etc/profile`文件，然后测试下java是否安装好。

   ```shell
   [root@vm3 jdk1.8.0_221]# javac -version
   javac 1.8.0_221
   [root@vm3 jdk1.8.0_221]# 
   
   ```

## zk安装

[下载](https://archive.apache.org/dist/zookeeper/zookeeper-3.4.14/)好zookeeper之后，解压，例如：

```shell
[root@vm3 conf]# pwd
/root/zookeeper/zookeeper-3.4.14/conf
[root@vm3 conf]# ls
configuration.xsl  log4j.properties  zoo_sample.cfg
[root@vm3 conf]# 


```

上面就是我在vm3机器上的zk目录。

关注下conf目录下的配置文件，其中`zoo_sample.cfg`是官方提供的一个配置文件的例子。

我们拷贝一份，重新命名一个zoo.cfg文件。只有zoo.cfg文件才会被zk识别为zk启动时加载的配置文件。

```shell
[root@vm3 conf]# cp zoo_sample.cfg zoo.cfg	
```

官方默认模板配置文件内容如下：

```shell
[root@vm3 conf]# cat zoo.cfg |grep -v '#'
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/tmp/zookeeper
clientPort=2181
[root@vm3 conf]# 


```

zookeeper服务器会产生三类日志：事务日志、快照日志和log4j日志。

- 事务日志路径，dataLogDir
- 快照日志路径，dataDir

dataDir放的是内存数据结构的snapshot（快照），这个是必须配置的。

dataLogDir，如果配置文件没有提供的话，那么就使用dataDir的路径。

dataLogDir里放的是顺序日志(WAL)。



## zk配置文件

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/root/zookeeper/zkdata
dataLogDir=/root/zookeeper/zklogdata
clientPort=2181
server.1=vm1:2888:3888
server.2=vm2:2888:3888
server.3=vm3:2888:3888
```

上面是我测试用的配置文件。

- clientPort，表示zk监听客户端连接的port。默认是2181端口
- server.x=host:portA:portB，表示zk集群的各个节点。其中x是一个数字, 表示这是第几号server。host是该server所在机器的IP地址，portA配置该server和集群中的leader交换消息所使用的端口，leader肯定会监听portA这个端口，其他follower与leader交换信息时，通过这个端口和leader进行通信。 C配置选举leader时所使用的端口。



在每个zk节点的`dataDir`目录，还需要有一个`myid`文件，表示自己是第几号server。对应的就是上面配置文件中的`server.1`就是1号server，`server.2`就是2号server。

```shell
[root@vm2 zkdata]# ls
myid  version-2  zookeeper_server.pid
[root@vm2 zkdata]# pwd
/root/zookeeper/zkdata
[root@vm2 zkdata]# cat myid 
2
[root@vm2 zkdata]# 

	
```

上面就是vm2机器上的myid文件。

配置host文件。

```shell
[root@vm3 bin]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

172.16.77.128 vm1
172.16.77.138 vm2
172.16.77.139 vm3
[root@vm3 bin]# 

```



## 启动zk

```shell
[root@vm2 bin]# ./zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /root/zookeeper/zookeeper-3.4.14/bin/../conf/zoo.cfg                                           
Starting zookeeper ... STARTED
[root@vm2 bin]# 
```

可以通过`./zkServer.sh status`查看zk的状态信息。

```shell
[root@vm3 bin]# ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /root/zookeeper/zookeeper-3.4.14/bin/../conf/zoo.cfg                                           
Mode: follower
[root@vm3 bin]# 



```

启动日志是在bin目录下的zookeeper.out文件。

```shell
[root@vm3 bin]# pwd
/root/zookeeper/zookeeper-3.4.14/bin
[root@vm3 bin]# ls -l zookeeper.out 
-rw-r--r--. 1 root root 11588 9月   2 23:42 zookeeper.out
[root@vm3 bin]# 
	
```

## zookeeper提供了什么

简单地说，zookeeper = 文件系统 + 通知机制。

1. 文件系统
   Zookeeper维护一个类似文件系统的数据结构。
   ![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/zk1.png)

   每个子目录项如 NameService 都被称作为 znode，和文件系统一样，我们能够自由地增加、删除znode，在一个znode下增加、删除子znode，唯一的不同在于znode是可以存储数据的。

2. 通知机制

   客户端注册监听它关心的目录节点，当目录节点发生变化（数据改变、被删除、子目录节点增加删除）时，zookeeper会通知客户端。

## zookeeper的节点

zk有多种不同的节点。

- PERSISTENT-持久化目录节点
- PERSISTENT_SEQUENTIAL-持久化顺序编号目录节点
- EPHEMERAL-临时目录节点
- EPHEMERAL_SEQUENTIAL-临时顺序编号目录节点



## golang zookeeper客户端

推荐使用go-zookeeper。

## zookeeper版本号

每一个znode都有一个版本号，它随着每次数据变化而自增。

两个api操作`setData`和`delete`均以版本号作为入参，只有当传入参数的版本号与服务器上的版本号一致时，调用才会成功。当多个客户端对同一个zone进行操作时，版本的使用显得尤为重要。