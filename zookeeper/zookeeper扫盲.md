# zookeeper扫盲 #
## 是什么？ ##

zookeeper可以用来干啥？

引用[Apache官方](https://zookeeper.apache.org/)的解释，

> ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services

中文解释就是：

- 维护配置信息
- 名字服务
- 分布式同步（分布式锁）
- 集群管理

下面一一解释一下

### 维护配置信息 ###

分布式系统都有好多机器，比如我们在搭建hadoop的HDFS的时候，需要在一个主机器上（Master节点）配置好HDFS需要的各种配置文件，然后通过scp命令把这些配置文件拷贝到其他节点上，这样各个机器拿到的配置信息是一致的，才能成功运行起来HDFS服务。

Zookeeper提供了这样的一种服务：一种集中管理配置的方法，我们在这个集中的地方修改了配置，所有对这个配置感兴趣的都可以获得变更。这样就省去手动拷贝配置了，还保证了可靠和一致性。 

### 名字服务 ###
### 分布式锁 ###
### 集群管理 ###