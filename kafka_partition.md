# kafka分区（partition）和group #
## 原理图 ##

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/kafka_cluster.png)



## kafka版本

目前广泛使用的kafka版本是0.8.x，0.9.x和0.10.*。这三个版本对于consumer和consumer group来说都有很大的变化。

## Kafka的Partition

一个topic可以有多个Partition，具体有多少个Partition不受broker的限制。比如说kafka集群有3哥节点，也就是3个broker，某个topic是可以有4个、5个或者更多的Partition的。

一个topic有多个Partition有啥好处呢？

## consumer Group

consumer group是kafka提供的可扩展且具有容错性的消费者机制。

既然是一个组，那么组内必然可以有多个消费者或消费者实例(consumer instance)，它们共享一个公共的ID，即group ID。

组内的所有消费者协调在一起来消费订阅主题(subscribed topics)的所有分区(partition)。

当然，对于某个consumer group，每个分区只能由这个消费组内的一个consumer来消费。

个人认为，理解consumer group记住下面这三个特性就好了：

1. consumer group下可以有一个或多个consumer instance，consumer instance可以是一个进程，也可以是一个线程
2. group.id是一个字符串，唯一标识一个consumer group
3. consumer group下订阅的topic下的每个分区只能分配给某个group下的一个consumer（当然该分区还可以被分配给其他group）



### 消费者位置（consumer position）

消费者在消费的过程中需要记录自己消费了多少数据，即消费位置信息。

在Kafka中这个位置信息有个专门的术语：位移(offset)。

很多消息引擎都把这部分信息保存在服务器端(broker端)。这样做的好处当然是实现简单，但会有三个主要的问题：

1. broker从此变成有状态的，会影响伸缩性
2. 需要引入应答机制(acknowledgement)来确认消费成功
3. 由于要保存很多consumer的offset信息，必然引入复杂的数据结构，造成资源浪费

## 原理图说明 ##

一个topic可以配置好几个partition，producer产生的信息发送到不同的partition中，consumer接收数据是按照group来接收。kafka确保每个partition只能同***一个***group中的***同一个***consumer消费。

如果想要重复消费，那么需要其他的组来消费。Zookeeper中保存着每个topic下的每个partition在每个group中消费的offset。

新版kafka把这个offsert保存到了一个__consumer_offsert的topic下，这个__consumer_offsert 有50个分区，通过将group的id按照id%50取哈希值的值来确定要保存到那一个分区。

一个consumer可以消费多个topic么？

比如说上图中Consumer Group A的group Id 为1，那么Group A下的所有consumer（C1，C2）消费的所有topic
