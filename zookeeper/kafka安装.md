# kafka安装

## 安装zk

步骤参照另外一篇md

## 安装kafka

### 官网下载kafka

可以去国内的源下载，https://mirrors.tuna.tsinghua.edu.cn/apache/kafka/。

找个目录解压，

```shell
# root @ localhost in ~ [1:35:54]
$ pwd
/root

# root @ localhost in ~ [1:36:15]
$ ls -l kafka_2.13-2.6.1
总用量 56
drwxr-xr-x. 3 root root  4096 12月 11 03:37 bin
drwxr-xr-x. 2 root root  4096 2月   5 00:25 config
drwxr-xr-x. 2 root root  8192 2月   4 01:41 libs
-rw-r--r--. 1 root root 29975 12月 11 03:31 LICENSE
drwxr-xr-x. 2 root root   237 2月   5 01:29 logs
-rw-r--r--. 1 root root   337 12月 11 03:31 NOTICE
drwxr-xr-x. 2 root root    44 12月 11 03:37 site-docs

# root @ localhost in ~ [1:36:23]
$
# root @ localhost in ~ [1:38:47]
$ ls -l kafka
lrwxrwxrwx. 1 root root 16 2月   4 01:42 kafka -> kafka_2.13-2.6.1

# root @ localhost in ~ [1:38:52]
$
```

### 配置文件

先配置profile，编辑`/etc/profile`，

```shell
export KAFKA_HOME=/root/kafka
export PATH=$PATH:$KAFKA_HOME/bin
```

然后source一下。

编辑kafka配置文件，路径为

```shell
# root @ localhost in ~/kafka/config [1:41:27]
$ pwd
/root/kafka/config

# root @ localhost in ~/kafka/config [1:41:33]
$ ls -l server.properties
-rw-r--r--. 1 root root 6856 2月   5 00:25 server.properties

# root @ localhost in ~/kafka/config [1:41:37]
$
```

如果kafka有三个broker，那么需要将broker.id=0相应地更改。比如第一台是0，第二台是1，第三台是2.

```
# The id of the broker. This must be set to a unique integer for each broker.
broker.id=0

# root directory for all kafka znodes.
zookeeper.connect=vm1:2181,vm2:2181,vm3:2181

############################# Log Basics #############################

# A comma separated list of directories under which to store log files
log.dirs=/kafkadata
```

zookeeper的配置 `vm1:2181,vm2:2181,vm3:2181`是之前搭建好的zk地址。

`log.dirs`这个目录是kafka的数据目录。

kafka运行的日志放在`$KAFKA_HOME/logs`目录下。所以你可以提前在`$KAFKA_HOME`目录下建好`logs`目录，用于存放kafka运行日志。

```shell
# root @ localhost in ~/kafka/logs [1:44:09]
$ ls
controller.log        kafka-request.log             kafkaServer.out  server.log                state-change.log
kafka-authorizer.log  kafkaServer-gc.log.0.current  log-cleaner.log  server.log.2021-02-05-00

# root @ localhost in ~/kafka/logs [1:45:00]
$
```

**最重要的参数**

### 启动

配置好之后，可以分别启动各个broker。

```
$ kafka-server-start.sh -daemon $KAFKA_HOME/config/server.properties
```

脚本都在`$KAFKA_HOME/bin`目录下面。

### 停止

在各个broker机器上停止，

```
$ kafka-server-stop.sh
```

## 常用的脚本

kafka自带有一些脚本。

1. 拉取topic有哪些

  ```shell
  # root @ localhost in ~/kafka/bin [1:52:29] C:1
  $ ./kafka-topics.sh --list --zookeeper vm1:2181

  # root @ localhost in ~/kafka/bin [1:52:59]
  $
  ```
	当然对于新版kafka，也可以不指定zk，

  ```shell
  # root @ localhost in ~/kafka/bin [10:53:50]
  $ ./kafka-topics.sh --list --bootstrap-server vm1:9092
  victor
  wind

  # root @ localhost in ~/kafka/bin [10:56:06]
  $
  ```

2. 创建topic

  ```shell
  # root @ localhost in ~/kafka/bin [1:52:59]
$ ./kafka-topics.sh --zookeeper vm1:2181 --create --topic wind --partitions 3 --replication-factor 3
  Created topic wind.
  
  # root @ localhost in ~/kafka/bin [1:55:08]
$ ./kafka-topics.sh --list --zookeeper vm1:2181
  wind
  
  # root @ localhost in ~/kafka/bin [1:55:15]
  $
  ```
  
  当然对于新版本kafka，也可以不指定zk，比如，

  ```shell
  # root @ localhost in ~/kafka/bin [10:52:14]
  $ ./kafka-topics.sh --create --bootstrap-server vm1:9092 --replication-factor 3 --partitions 3 --topic victor
  Created topic victor.

  # root @ localhost in ~/kafka/bin [10:53:50]
  $
  ```



3. 查看topic信息

  ```shell
  # root @ localhost in ~/kafka/bin [1:56:38]
  $ ./kafka-topics.sh --zookeeper vm1:2181 --describe --topic wind
  Topic: wind     PartitionCount: 3       ReplicationFactor: 3    Configs:
          Topic: wind     Partition: 0    Leader: 0       Replicas: 0,1,2 Isr: 0,1,2
          Topic: wind     Partition: 1    Leader: 1       Replicas: 1,2,0 Isr: 1,2
          Topic: wind     Partition: 2    Leader: 2       Replicas: 2,0,1 Isr: 2,1

  # root @ localhost in ~/kafka/bin [1:56:41]
  $
  ```
对于新版本kafka，也可以不指定zk，

  ```shell
  # root @ localhost in ~/kafka/bin [10:56:06]
  $ ./kafka-topics.sh --describe --bootstrap-server vm1:9092 --topic wind
  Topic: wind     PartitionCount: 3       ReplicationFactor: 3    Configs: segment.bytes=1073741824
  Topic: wind     Partition: 0    Leader: 0       Replicas: 0,1,2 Isr: 1,0,2
  Topic: wind     Partition: 1    Leader: 1       Replicas: 1,2,0 Isr: 1,0,2
  Topic: wind     Partition: 2    Leader: 2       Replicas: 2,0,1 Isr: 1,0,2

  # root @ localhost in ~/kafka/bin [10:58:56]
  $ ./kafka-topics.sh --describe --bootstrap-server vm1:9092 --topic victor
  Topic: victor   PartitionCount: 3       ReplicationFactor: 3    Configs: segment.bytes=1073741824
  Topic: victor   Partition: 0    Leader: 2       Replicas: 2,1,0 Isr: 2,1,0
  Topic: victor   Partition: 1    Leader: 1       Replicas: 1,0,2 Isr: 1,0,2
  Topic: victor   Partition: 2    Leader: 0       Replicas: 0,2,1 Isr: 0,2,1

  # root @ localhost in ~/kafka/bin [10:59:03]
  $
  ```



4. 生产消息

  ```shell
  $ ./kafka-console-producer.sh --topic wind --broker-list vm1:9092
  >abc
  >def
  >ghi
  ```

	也可以这样，

  ```shell
  # root @ localhost in ~/kafka/bin [13:14:47]
  $ ./kafka-console-producer.sh --topic wind --bootstrap-server vm1:9092
  >aaaaaaa
  >bbbbbbb
  ```

	可能会失败，这时候需要改下kafka的配置文件，

  ```shell
  ############################# Socket Server Settings #############################

  host.name=vm1

  listeners=PLAINTEXT://vm1:9092
  advertised.listeners=PLAINTEXT://vm1:9092

  ```

	注意上面的vm1，需要同时在vm2和vm3上改，将vm1n iou  替换为对应的vm2和vm3。

	然后stop kafka，再重启。

5. 消费消息

   ```shell
   # root @ localhost in ~/kafka/bin [13:18:21]
   $ ./kafka-console-consumer.sh --bootstrap-server vm1:9092 --topic wind --from-beginning
   you
   bbbbbbb
   abc
   ghi
   wind
   beautiful
   aaaaaaa
   ccccccc
   def
   dzf
   are
   a
   girl!
   ```

6. 消费组消费消息
   我们可以使用官方的一些脚本测试consumer-group的概念。

   第一个consumer，如下：

   ```
   # root @ localhost in ~/kafka/bin [19:25:47] C:130
   $ ./kafka-console-consumer.sh --bootstrap-server vm2:9092 --topic wind --from-beginning --group testgroup
   fdsa
   fasd
   ```

   第二个consumer，如下

   ```
   [root@vm3 bin]# ./kafka-console-consumer.sh --bootstrap-server
   vm2:9092 --topic wind --from-beginning --group testgroup
   abc
   def
   fdafd
   dfasdf
   fdasd
   fdas
   fsd
   ```

   我们可以看到消费组`testgroup`有2个消费者。2个消费者同时消费 topic `wind`。

   我们还可以使用`kafka-consumer-groups.sh`这个脚本查询消费组的信息。

   ```
   [root@vm2 bin]# ./kafka-consumer-groups.sh --bootstrap-server vm2:9092 --group testgroup --describe
   
   GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                               HOST            CLIENT-ID
   testgroup       wind            0          9               9               0               consumer-testgroup-1-80a72bad-1209-4a92-a088-420e3242ab20 /192.168.6.162  consumer-testgroup-1
   testgroup       wind            1          9               9               0               consumer-testgroup-1-80a72bad-1209-4a92-a088-420e3242ab20 /192.168.6.162  consumer-testgroup-1
   testgroup       wind            2          4               4               0               consumer-testgroup-1-c80d3325-1856-41ea-ac6d-2f7b16d1677d /192.168.6.160  consumer-testgroup-1
   [root@vm2 bin]#
   ```

   可以看到，这个消费组，有2个消费者，分别是`consumer-testgroup-1-80a72bad-1209-4a92-a088-420e3242ab20`和`consumer-testgroup-1-c80d3325-1856-41ea-ac6d-2f7b16d1677d`。

   其中，`consumer-testgroup-1-c80d3325-1856-41ea-ac6d-2f7b16d1677d`承担`partition-2`的消费，`consumer-testgroup-1-80a72bad-1209-4a92-a088-420e3242ab20`承担`partition-0`和`partition-1`这2个partition的消费。对于1个consumer-group的不同消费者来说，他们不能消费1个topic的同1个partition。

   1个消费组下的consumer消费到了哪个offset呢？这些信息都是有保存在kafka中的。所以你如果重启`kafka-console-consumer.sh`脚本，但是使用同1个groupid，那么他不会从头开始消费，而是从上次消费到的offset开始。