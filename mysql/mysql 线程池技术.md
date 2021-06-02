# mysql 线程池技术

## 架构图

![](https://raw.githubusercontent.com/ernest-dzf/docs/master/pic/mysql_thread_pool.jpg)

从架构图中可以看到Thread Pool由一个Timer线程和多个Thread Group组成，而每个Thread Group又由两个队列、一个listener线程和多个worker线程构成。

- **队列**
  用来存放待执行的任务，分为高优先级队列和低优先级队列，高优先级队列的任务会优先被处理。
  什么任务会放在高优先级队列呢？
  事务中的语句会放到高优先级队列中，比如一个事务中有两个update的SQL，有1个已经执行，那么另外一个update的任务就会放在高优先级中。
  这里需要注意，如果是非事务引擎，或者开启了Autocommit的事务引擎，都会放到低优先级队列中。
  还有一种情况会将任务放到高优先级队列中，如果语句在低优先级队列停留太久，该语句也会移到高优先级队列中，防止饿死。

- **listener线程**
  listener只是一种角色，每个线程的角色可以是listener或者是worker

- **worker线程**
  worker线程是真正干活的线程，只是一种角色

- **Timer线程**
  Timer线程是用来周期性检查group是否处于阻塞状态，当出现阻塞的时候，会通过唤醒线程或者新建线程来解决。

  

工作线程有如下几种状态

- **活跃状态**，当工作线程处于正在处理任务且未被阻塞的状态，这意味着工作线程将会消耗CPU，增加系统的负载。如果工作线程将自己设置为listener，则不算进线程组的活跃线程状态数。
- **空闲状态**，由于没有任务处理而处于空闲状态。
- **等待状态**，如果工作线程在执行命令的过程中由于IO、锁、条件、sleep等需要等待，则线程池将被通知，并且将这些工作线程记作等待状态。

在线程组中，关于线程的计数有如下关系：

```
thread_count = active_thread_count + waiting_thread_count + waiting_threads.length + listener.length
```

thread_count代表线程组中的总线程数，active_thread_count代表当前正在工作且未被阻塞的线程数，waiting_thread_count代表的是工作线程执行任务的过程中被阻塞的个数，而waiting_threads代表空闲线程列表。

在MySQL线程池中，线程组中busy的线程数是active_thread_count与waiting_thread_count的总和，因为这些线程此时都不能处理新的任务，因此被认为是繁忙的。

如果处于busy状态的线程数大于一定值，则线程组被认为是太繁忙（too many active）了，这会用于决策普通优先级的任务是否能得到及时的处理，这个值被定义为，`thread_pool_oversubscribe + 1`。

## 相关变量或者状态变量

- **thread_handling**
  该参数表示配置的是什么线程模型。

  ```mysql
  MariaDB [(none)]> show variables like '%thread_handling%';
  +-----------------+-----------------+
  | Variable_name   | Value           |
  +-----------------+-----------------+
  | thread_handling | pool-of-threads |
  +-----------------+-----------------+
  1 row in set (0.00 sec)
  
  MariaDB [(none)]> 
  
  ```

  如上表示是 线程池 模型。

- **thread_pool_size**
  这个参数指的是线程组大小，默认是CPU核心数，线程池初始化的时候会根据这个数字来生成线程组。
  云上默认为 24。

  ```mysql
  MariaDB [(none)]> show variables like '%thread_pool_size%';                                                                         
  +------------------+-------+
  | Variable_name    | Value |
  +------------------+-------+
  | thread_pool_size | 24    |
  +------------------+-------+
  1 row in set (0.01 sec)
  
  MariaDB [(none)]> 
  
  ```

  

- **thread_pool_stall_limit**
  该参数设置timer线程的检测group是否异常的时间间隔，默认为500ms。

- **thread_pool_oversubscribe**
  这个参数控制每个group的最大并发线程数，每个group的最大并发线程数为`thread_pool_oversubscribe+1`个。若对应的group达到了最大的并发线程数，则对应的连接就需要等待。
  这里的`+1`考虑的是listener线程。

- **thread_pool_max_threads**
  这个参数用来限制线程池最大的线程数，超过该限制后将无法再创建更多的线程。

- **thread_pool_idle_timeout**
  worker线程最大空闲时间，默认为60秒，超过限制后会退出。

- **thread_pool_high_prio_mode**
  高优先级队列的控制参数。

  ```mysql
  MySQL [(none)]> show variables like '%thread_pool_high_prio_mode%';
  +----------------------------+--------------+
  | Variable_name              | Value        |
  +----------------------------+--------------+
  | thread_pool_high_prio_mode | transactions |
  +----------------------------+--------------+
  1 row in set (0.02 sec)
  
  MySQL [(none)]> 
  
  ```

  有三个值，`transactions\statements\none`。

  - transactions，对于已经启动事务的语句放到高优先级队列中，不过还取决于后面的thread_pool_high_prio_tickets参数
  - statements，这个模式所有的语句都会放到高优先级队列中，不会使用到低优先级队列
  - none，这个模式不使用高优先级队列

- **thread_pool_high_prio_tickets**
  该参数控制每个连接的任务最多多少次被放入高优先级队列中，默认为4294967295，注意这个参数只有在thread_pool_high_prio_mode为transactions的时候才有效果。
  
- **Threads_connected**
  记录连接的线程数，跟`show processlist`结果相同，表示当前连接数。包括Sleep状态的。

- **Threads_running**
  表示busy的线程数目？
  
  >Threads connected means the total number of client processes (threads) connected to the database server. This includes the count for threads running.
  
  > Thread running means the total number of client processes (threads) currently executing on the database server. The server is holding these connections while the client is waiting for a reply. These thread may be consuming IO/CPU, while others may do nothing while waiting for a table lock to be released. When the database is finished executing the thread, the client gets a reply, and the thread is changed from status "running" to "connected".

## 

## 参考文章

1. https://blog.csdn.net/u012662731/article/details/54375137
2. https://blog.csdn.net/Stubborn_Cow/article/details/50246043
3. https://dbaplus.cn/news-11-1989-1.html
4. https://my.oschina.net/andylucc/blog/820624