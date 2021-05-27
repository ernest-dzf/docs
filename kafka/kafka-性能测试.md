# kafka 性能测试

Kafka系统提供了测试工具kafka-producer-perf-test.sh和kafka-consumer-perf-test.sh，通过该工具可以对生产者性能和消费者性能进行测试，获取一组最佳的参数值，进而提升生产者的发送效率和消费者的读取效率。

```shell
[root@VM_144_188_centos /data/victor/kafka_2.10-0.10.0.0/bin]# ls
connect-distributed.sh     kafka-consumer-offset-checker.sh     kafka-replica-verification.sh   kafka-verifiable-producer.sh
connect-standalone.sh      kafka-consumer-perf-test.sh          kafka-run-class.sh              windows
kafka-acls.sh              kafka-mirror-maker.sh                kafka-server-start.sh           zookeeper-security-migration.sh
kafka-configs.sh           kafka-preferred-replica-election.sh  kafka-server-stop.sh            zookeeper-server-start.sh
kafka-console-consumer.sh  kafka-producer-perf-test.sh          kafka-simple-consumer-shell.sh  zookeeper-server-stop.sh
kafka-console-producer.sh  kafka-reassign-partitions.sh         kafka-topics.sh                 zookeeper-shell.sh
kafka-consumer-groups.sh   kafka-replay-log-producer.sh         kafka-verifiable-consumer.sh
[root@VM_144_188_centos /data/victor/kafka_2.10-0.10.0.0/bin]# ls -l kafka-producer-perf-test.sh 
-rwxrwxrwx 1 root users 959 May 18  2016 kafka-producer-perf-test.sh
[root@VM_144_188_centos /data/victor/kafka_2.10-0.10.0.0/bin]# ls -l kafka-cons
kafka-console-consumer.sh         kafka-consumer-groups.sh          kafka-consumer-perf-test.sh       
kafka-console-producer.sh         kafka-consumer-offset-checker.sh  
[root@VM_144_188_centos /data/victor/kafka_2.10-0.10.0.0/bin]# ls -l kafka-consumer-perf-test.sh 
-rwxrwxrwx 1 root users 948 May 18  2016 kafka-consumer-perf-test.sh
[root@VM_144_188_centos /data/victor/kafka_2.10-0.10.0.0/bin]# 

```

比如，

```
./kafka-producer-perf-test.sh  --topic benchmark-1 --num-records 1000000000 --record-size 1000  --producer-props bootstrap.servers=100.119.167.50:6882 --throughput -1
```

参数含义如下：

|                 参数                  |                         含义                          |
| :-----------------------------------: | :---------------------------------------------------: |
|                --topic                |               指定生产者发送消息的主题                |
|             --num-records             |               测试时发送消息的总记录数                |
|             --throughput              |       最大消息吞吐量，为-1的话，表示不设置上限        |
|           --producer-props            | 通过键值对的方式指定配置属性，多组配置属性用空格分隔  |
|             --record-siez             |                   每条消息字节大小                    |
| bootstrap.servers=100.119.167.50:6882 | 对应--producer-props说的键值对，表示kafka的broker地址 |
|                                       |                                                       |
|                                       |                                                       |
|                                       |                                                       |



测试消费，可以这样，

```
./kafka-consumer-perf-test.sh --broker-list 9.139.0.189:9092 --messages 10000000 --topic victortest-2 --new-consumer --fetch-size 8388608
```

