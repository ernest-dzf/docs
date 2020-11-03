# mysql 线程池技术

## 相关变量

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

## 参考文章

1. https://blog.csdn.net/u012662731/article/details/54375137
2. https://blog.csdn.net/Stubborn_Cow/article/details/50246043
3. https://dbaplus.cn/news-11-1989-1.html