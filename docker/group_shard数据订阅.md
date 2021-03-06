# 预设前提条件

1. 多分片
2. 每个分片含有主键
3. 含有shard key
4. 每个shard的表结构一样
5. DDL是非原子的，有可能某个分片DDL失败，但是用户不知道
6. 不保证订阅过程中数据一致性，只保证最终数据一致性
7. 数据订阅输出数据中要附加表结构信息（体现形式？）
8. 输出数据不能有大量冗余binlog信息，能做到不多一条，也不少一条是最好
9. 对于分片表，认为表结构不一致是不正常的，如果出现，同步任务受到影响是可接受的，但要有方法去修
10. 只考虑分布式percona，支持xa事务的
11. 订阅起点，以某个分布式事务为起点

## schema获取的问题

采用 **导出基线，binlog重放补偿**的方法。

在导出基线的过程中，各个task分别去做表结构导出。

这里有问题是：**我们在时间t1，获取到位点是pos-A，然后在时间点t2，show create table 得到表结构，如何确保在t1-t2时间点，用户没有去做DDL？**



在 **导出基线**的过程，

对于group shard，前提条件是多个分片的表结构要一致。（这个在校验阶段可以保证）

但是用户做的DDL**不是原子的**，在导schema的过程中，用户可能做ddl，可能出现多个分片表结构不一致的问题。



针对**DDL非原子的问题**，用户做了DDL，在set-1成功了，在set-2失败了，

**导完表结构之后，各个task需要再次check一下大家的表结构是否一样，不一样就报错。**



开启分布式事务，作为各个set的订阅起点。

各个set 应用 binlog 到对应的 binlog 位点。得到各自的schema结构，然后比对是否有差异。





上述策略可以解决如下问题：

1. 订阅起点问题
2. schema一致性问题
3. 表结构获取的时候，加ftwrl导致业务受影响问题
4. 极端情况下，用户ddl 某个分片失败，导致表结构不一致问题，我们提前把错误抛出来



## 增量过程中DDL

前提：增量过程中的DDL必须是做到各个分片都成功执行。如果出现DDL某个分片是比，预期任务要报错，修复DDL之后，任务可以继续



某个task如果收到DDL EVENT，立即停住，并将本线程的和DDL相关的之前的event都投递完毕，并告知其他线程自己收到这个DDL。



其他线程预期收到这个通知之后，会继续DML的投递，直到也收到相同的DDL。



这里需要有1个超时机制，比如1小时都没有收到，需要告警出来。

## 针对水平扩容

1. 需要感知新加分片什么时候买好了，可以通过 show prxy status感知到。2s show 一次，然后针对新set，change master 往前一段值，拍脑袋定一个就好。根据binlog 时间戳做正确性判断。
2. 新set binlog 接管肯定有重复binlog，根据server id 判断binlog来源，如果是其他shard的，过滤掉。



以上解决add shard binlog 交叉问题。



针对冗余数据的删除，

alter table drop partition 直接过滤

## 针对垂直扩容

show proxy status

## 针对幂等

要求有主键

当一个表的约束定义除了包含主键外，还有唯一索引，保证相同唯一索引的事件按照顺序来执行。没有并发的话，不用考虑

## 针对二级分区

不支持。

## 针对单表

用户如果存在单表，在扩容之后，新加set会有单表的drop table binlog，过滤掉。

初期可以不支持单表。

## 待确认的点

1. kafak 谁维护