- 支持在创建topic时指定partition放在哪些broker上
- 消费者可以指定分区进行消费
- kafka消费端，StickyAssignor参数，解决Rebalance问题
- 分区reassign
- 分区available是指？
- 1个topic的某个partition的follower副本在哪个broker可以指定
- 如何给kafka扩容，比如添加新的broker机器
- 将某个topic的某个分区下掉
- 使用带有回调通知的kafka api
- 当且仅当request.required.acks参数设置为-1时，min.insync.replicas参数才生效。min.insync.replicas这个参数设定ISR中的最小副本数是多少
- kafka是在落地刷盘之后，同步副本成功后，才能会被消费吗？其实，有可能在落盘之前就被消费了。能否被消费不是看是否flush到磁盘，而是看leader副本的高水位是否越过了该条消息
- 幂等producer
- 创建topic指定分区个数，指定分区副本个数
- isr是强同步么？
- 不支持减少topic的分区，但是支持减少分区的副本个数
- kafka只能异步发送消息么？
- sarama的回调函数，如果由于网络原因，设置的retries>1，sarama自己回调了，那么还会触发回调呢？回调是每次request都会触发么？
- isr副本的要求是？
- 消息有可能在落盘之前就被消费了，其实，有可能在落盘之前就被消费了。能否被消费不是看是否flush到磁盘，而是看leader副本的高水位是否越过了该条消息
- 老师，你好。仔细阅读文稿后，仍有一些困惑
  1、如果只用 send()方法（fire and forget）， 即使配置retries，producer也是不知道消息状态，是不会重试的。所以说配置retries，要搭配send(msg, callback)，这么理解正确么？
  2、配置了retries, producer是怎么知道哪条消息发送失败了，然后重试
- 作者回复: 1. 不是。如果配置了retries，即使调用send(msg)也是会重试的。这是Kafka producer自己实现的机制，不需要用户干预
  \2. Broker发送response给producer，里面会保存error信息以及那个(些)batch出错了
- 老师好，有个问题，就是：retries 和 send里面的callback，是什么关系？因为有说retries是kafka自动重试的次数，那么还要callback干吗，callback的意义在哪里呢？ 如果一定要坚持用send(callback)api，那么retries是用来干吗的呢？ 这两者之间的关系是什么呢？谢谢。作者回复: callback可以处理消息发送之后的逻辑，不一定就是失败的逻辑。retries是预防那种瞬时错误的，比如网络抖动这种问题，让Kafka自动重试一下会比较方便不是吗
- max.poll.interval.ms
- offset.auto.reset=lastest，auto.offset.reset
- 在“producer.send(msg, callback)的callback方法中重试，和设置retries参数重试，会不会冲突？2个都设置以哪个为准？作者回复: 不冲突。对于可重试的错误，retries才会触发，否则直接 进入到callback
- acks=all 分区副本的个数为3，producer将消息发送给Leader分区后，两个Follower分区还未同步，或者由于网络延时问题迟迟没有同步，这时消息者是否可以消费Leader分区上的这条消息，... 不可以！！
- 操作系统的page cache
- 老师，我有一个问题想不明白。ack参数是设置在生产者上的，不同的生产者可以有不同的参数值，那是不是同一个主题不同的生产者"已提交"的定义是不一样的？这个定义是以生产者为准的？还是以服务器设置为准的？这么看来，是以生产者为准
- retries如果大于3，第一次失败，第二次还是失败，第三次成功了，是否会通过callback告诉producer者失败呢？
- kafka  paxos或者raft
- kafka 零拷贝
- 老师我想问一下，如果Broker端一定会解压缩，那么一定会将数据从内核态拷贝到用户态，那么Kafka是怎么实现内核态的零拷贝。不是说解压缩就一定会丧失Zero Copy。因版本转化而导致消息格式要被重构的场景才会丧失Zero Copy。否则，只是解压缩，数据依然在页缓存中啊
- kafka拦截器
- max.poll.interval.ms
- 可以删除consumer group
- 那我如果有100个分区，100个同组消费者，在启动这100个消费者过程中会发生100次rebalace吗。目前不会！！
- 老师，我遇到一个生产上的问题：一个消费组(大概有300多个消费者)订阅了3个topic，如topic1 ,tipic2，topic3 ， 增加一个topic4，然后在这个消费组里面增加消费者来消费topic4的数据。
  问题：是整个消费组发生重平衡吗？还是只是订阅topic4的消费者发生重平衡？嗯，300+个consumer都会重平衡。。。
- kafka coordinator controller
- interceptor
- advertise. listeners
- 何时更新isr
- 每个broker都有协调者
- xfs