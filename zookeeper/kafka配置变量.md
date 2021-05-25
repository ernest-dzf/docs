# kafka配置变量

## heartbeat.interval.ms

> The expected time between heartbeats to the consumer coordinator when using Kafka's group management facilities. Heartbeats are used to ensure that the consumer's session stays active and to facilitate rebalancing when new consumers join or leave the group. The value must be set lower than `session.timeout.ms`, but typically should be set no higher than 1/3 of that value. It can be adjusted even lower to control the expected time for normal rebalances.

控制重平衡的通知频率。

如果你想要消费者实例更迅速地得到通知，那么就可以给这个参数设置一个非常小的值，这样消费者就能更快地感知到重平衡已经开启了。

消费者有1个心跳线程（heartbeat thread，along with user thread which invokes `Poll` function in the same process）。消费者会每隔`heartbeat.interval.ms`，给coordinator发送心跳。这个值一般会比`session.timeout.ms`小，建议为`session.timeout.ms`的`1/3`。

coordinator收到consumer的心跳request之后，通过Response告诉consumer发起Rebalance。

## session.timeout.ms

> The timeout used to detect client failures when using Kafka's group management facility. The client sends periodic heartbeats to indicate its liveness to the broker. If no heartbeats are received by the broker before the expiration of this session timeout, then the broker will remove this client from the group and initiate a rebalance. Note that the value must be in the allowable range as configured in the broker configuration by `group.min.session.timeout.ms` and `group.max.session.timeout.ms`.

这个也是给心跳线程用的。如果超过`session.timeout.ms`时间，coordinator没有收到consumer的心跳，那么coordinator认为consumer已经挂了，就会将这个consumer从group中移除，并发起Rebalance。

## max.poll.interval.ms

> The maximum delay between invocations of poll() when using consumer group management. This places an upper bound on the amount of time that the consumer can be idle before fetching more records. If poll() is not called before expiration of this timeout, then the consumer is considered failed and the group will rebalance in order to reassign the partitions to another member. For consumers using a non-null `group.instance.id` which reach this timeout, partitions will not be immediately reassigned. Instead, the consumer will stop sending heartbeats and partitions will be reassigned after expiration of `session.timeout.ms`. This mirrors the behavior of a static consumer which has shutdown.

Consumer消费时间过长，导致超过了max.poll.interval.ms时间，Consumer主动发起离开Group请求。

`max.poll.interval.ms`控制2次连续的`poll()`请求之间的时间间隔。如果2次`poll()`的间隔超过了`max.poll.interval.ms`，consumer client会**主动**向coordinator发起LeaveGroup请求，触发rebalance；然后consumer重新发送JoinGroup请求。

参考：https://www.cnblogs.com/sniffs/p/13205196.html

