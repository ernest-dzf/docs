# kafka分区（partition）和group #
## 1. 原理图 ##

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/kafka_cluster.png)

## 2. 原理图说明 ##

一个topic可以配置好几个partition，producer产生的信息发送到不同的partition中，consumer接收数据是按照group来接收。kafka确保每个partition只能同***一个***group中的***同一个***consumer消费。

如果想要重复消费，那么需要其他的组来消费。Zookeeper中保存这每个topic下的每个partition在每个group中消费的offset。

新版kafka把这个offsert保存到了一个__consumer_offsert的topic下，这个__consumer_offsert 有50个分区，通过将group的id按照id%50取哈希值的值来确定要保存到那一个分区。

一个consumer可以消费多个topic么？

比如说上图中Consumer Group A的group Id 为1，那么Group A下的所有consumer（C1，C2）消费的所有top

ic