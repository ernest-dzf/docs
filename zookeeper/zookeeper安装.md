

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



![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/zkversion.jpg)

比如上图，客户端c1对znode `/config`写入了一些信息，如果另外一个客户端c2同时更新了这个znode，此时c1的版本号已经过期，c1调用`setData`一定会失败。

## zk提供的API

zk暴露了一些API，以让应用实现自己的原语。

- create /path data，创建一个名为`/path`的znode节点，并包含数据data
- delete /path，删除名为`/path`的znode
- exists /path，检查是否存在名为`/path`的节点
- setData /path data，设置名为`/path`的znode的数据为data
- getData /path，返回名为`/path`节点的数据信息
- getChildren /path，返回所有`/path`节点的所有子节点列表
- ……



zookeeper提供的api不止上面这些，详细的可以看官方的[文档](https://zookeeper.apache.org/doc/r3.4.6/api/org/apache/zookeeper/ZooKeeper.html)。

客户端与zk集群通信时，采用zk规定的协议。可以理解为zk通过识别解析tcp的payload，来确定是什么请求。

具体的请求有对应的opcode，如下：

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/zkopcode.png)

## zk的通知机制

客户端在获取zk集群的znode数据时，可以不必去轮询这个znode 的数据。

客户端可以向zk注册需要接受通知的znode，通过对znode设置监视点来接收通知。

监视点是一个单次触发的操作，意即监视点会触发一个通知。

为了接收多个通知，客户端必须在每次通知之后，设置一个新的监视点。

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/zknotify.jpg)

但是这种通知机制会丢失事件么？

**是可能的**！

怎么避免呢？

用户可以在设置新的监视点前，读取zk的状态，这样就不会错误任何变更。

读取zk和设置监视点是原子的操作（调用zk暴露的api，比如getData）。

zookeeper的getData()，getChildren()和exists()方法都可以注册watcher监听。

zookeeper的watch监听有以下特性：

- 一次性触发（one-time trigger） 
  当数据改变的时候，那么一个Watch事件会产生并且被发送到客户端中。但是客户端只会收到一次这样的通知，如果以后这个数据再次发生改变的时候，之前设置Watch的客户端将不会再次收到改变的通知，因为Watch机制规定了它是一个一次性的触发器。
- 发送给客户端（Sent to the client）
  这个表明了watch的通知事件是从服务器发送给客户端的，是异步的。不同的客户端收到的watch事件的时间可能不同，但是ZooKeeper有保证：当一个客户端在看到Watch事件之前是不会看到结点数据的变化的。
  例如：A=3，此时在上面设置了一次Watch，如果A突然变成4，那么客户端会先收到Watch事件的通知，然后才会看到A=4。（**不同的客户端看到4的时间也不一样？？？**）
- 被设置了watch的数据（The data for which the watch was set） 
  这是指节点发生变动的不同方式。
  你可以认为ZooKeeper维护了两个watch列表：data watch和child watch。getData()和exists()设置data watch，而getChildren()设置child watch。**或者，可以认为watch是根据返回值设置的**。getData()和exists()返回节点本身的信息，而getChildren()返回 子节点的列表。
  因此，setData()会触发znode上设置的data watch（如果set成功的话）。一个成功的 create() 操作会触发被创建的znode上的数据watch，以及其父节点上的child watch。而一个成功的 delete()操作将会同时触发一个znode的data watch和child watch（因为这样就没有子节点了），同时也会触发其父节点的child watch。

当client重新连接时，如果需要的话，所有先前注册过的watch，都会被重新注册。通常这是完全透明的。

只有在一个特殊情况下，watch可能会丢失：对于一个未创建的znode的exist watch，如果在客户端断开连接期间被创建了，并且随后在客户端连接上之前又删除了，这种情况下，这个watch事件可能会被丢失。 

### zookeeper对watch提供的保障

对于watch，ZooKeeper提供了这些保障：

- Watch与其他事件、其他watch以及异步回复都是有序的。 ZooKeeper客户端库保证所有事件都会按顺序分发。
- 客户端会保障它在看到相应的znode的新数据之前接收到watch事件。
- 从ZooKeeper接收到的watch事件顺序一定和ZooKeeper服务所看到的事件顺序是一致的。

## 两个客户端同时对znode进行操作？

有版本号。

## 会话

当客户端通过某一个特定语言套件来创建一个zookeeper句柄时，他就会通过服务建立一个会话。

当会话无法于当前连接的服务器继续通信时，会话就可能转移到另外一台服务器上。这个转移过程是透明的。

客户端提供给zk的所有操作，均需要关联在一个会话上。当一个会话因某种原因终止时，这个会话期间创建的临时节点将会消失。

**通常一个客户端只会打开一个会话。**会话提供了顺序保障，同一个会话中的请求会以FIFO顺序执行。

### 会话的生命周期

会话状态转移如下：

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/session_state.jpg)

## zkCli.sh的操作

### 创建节点

`create [-s] [-e] path data acl`

- -s，表示创建顺序节点
- -e，表示创建临时节点

### 删除节点

`delete path [version]`

## zookeeper的事件类型

我们知道客户端可watch某一个事件，那么zk有哪些可以watch的事件呢？

data events如下：

| Trigger                                                      | Event Type                              | Watches                                                      |
| ------------------------------------------------------------ | --------------------------------------- | ------------------------------------------------------------ |
| [ZooKeeper.create](http://zookeeper.apache.org/doc/r3.4.8/api/org/a/pache/zookeeper/ZooKeeper.html#create-java.lang.String-byte:A-java.util.List-org.apache.zookeeper.CreateMode-) | Watcher.Event.EventType.NodeCreated     | [ZooKeeper.exists](http://zookeeper.apache.org/doc/r3.4.8/api/org/apache/zookeeper/ZooKeeper.html#exists-java.lang.String-boolean-)<br>[ZooKeeper.getData](http://zookeeper.apache.org/doc/r3.4.8/api/org/apache/zookeeper/ZooKeeper.html#getData-java.lang.String-boolean-org.apache.zookeeper.AsyncCallback.DataCallback-java.lang.Object-) |
| [ZooKeeper.setData](http://zookeeper.apache.org/doc/r3.4.8/api/org/apache/zookeeper/ZooKeeper.html#setData-java.lang.String-byte:A-int-) | Watcher.Event.EventType.NodeDataChanged | [ZooKeeper.getData](http://zookeeper.apache.org/doc/r3.4.8/api/org/apache/zookeeper/ZooKeeper.html#getData-java.lang.String-boolean-org.apache.zookeeper.AsyncCallback.DataCallback-java.lang.Object-) |
| [ZooKeeper.delete](http://zookeeper.apache.org/doc/r3.4.8/api/org/apache/zookeeper/ZooKeeper.html#delete-java.lang.String-int-) | Watcher.Event.EventType.NodeDeleted     | [ZooKeeper.exists](http://zookeeper.apache.org/doc/r3.4.8/api/org/apache/zookeeper/ZooKeeper.html#exists-java.lang.String-boolean-) |
|                                                              |                                         |                                                              |
|                                                              |                                         |                                                              |

child events 如下：

| Trigger                                                      | Event Type                                  | Watches                                                      |
| ------------------------------------------------------------ | ------------------------------------------- | ------------------------------------------------------------ |
| [ZooKeeper.create](http://zookeeper.apache.org/doc/r3.4.8/api/org/apache/zookeeper/ZooKeeper.html#create-java.lang.String-byte:A-java.util.List-org.apache.zookeeper.CreateMode-) | Watcher.Event.EventType.NodeChildrenChanged | [ZooKeeper.getChildren](http://zookeeper.apache.org/doc/r3.4.8/api/org/apache/zookeeper/ZooKeeper.html#getChildren-java.lang.String-boolean-) |
| [ZooKeeper.delete](http://zookeeper.apache.org/doc/r3.4.8/api/org/apache/zookeeper/ZooKeeper.html#delete-java.lang.String-int-) | Watcher.Event.EventType.NodeChildrenChanged | [ZooKeeper.exists](http://zookeeper.apache.org/doc/r3.4.8/api/org/apache/zookeeper/ZooKeeper.html#exists-java.lang.String-boolean-)<br>[ZooKeeper.getChildren](http://zookeeper.apache.org/doc/r3.4.8/api/org/apache/zookeeper/ZooKeeper.html#getChildren-java.lang.String-boolean-) |

session events如下：

