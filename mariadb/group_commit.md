# mysql group commit

本文提到的是针对mysql或者percona的，mariadb 的可能不一样。

我们使用mysqlbinlog查看binlog文件时候，

```
# at 46128225
#200705 16:27:16 server id 1424200933  end_log_pos 46128286     GTID    last_committed=128999   sequence_number=129000
SET @@SESSION.GTID_NEXT= 'aaad7db1-aed7-11ea-a392-5cb9018f72f8:2303964'/*!*/;

# at 46128286
#200705 16:27:16 server id 1424200933  end_log_pos 46128366     Query   thread_id=33    exec_time=0     error_code=0
SET TIMESTAMP=1593937636/*!*/;
BEGIN
/*!*/;

# at 46128366
#200705 16:27:16 server id 1424200933  end_log_pos 46128559     Query   thread_id=33    exec_time=0     error_code=0
SET TIMESTAMP=1593937636/*!*/;
replace into SysDB.StatusTableForHb set tid='7f8dee7fc700',ts = from_unixtime(1593937636),ip='10.112.84.227',port=4069
/*!*/;

# at 46128559
#200705 16:27:16 server id 1424200933  end_log_pos 46128586     Xid = 12530526
COMMIT/*!*/;

```

上面是一个完整的事务的binlog，对应的其实有4个binlog event。分别是，GTID_EVENT，QUERY_EVENT，QUERY_EVENT，XID_EVENT。



GTID_EVENT表示一个事务的开始，我们可以看到，`SET @@SESSION.GTID_NEXT= 'aaad7db1-aed7-11ea-a392-5cb9018f72f8:2303964'/*!*/;`，表示当前这个事务的GTID是`aaad7db1-aed7-11ea-a392-5cb9018f72f8:2303964`。

然后紧跟着的是一个`BEGIN`。`BEGIN`其实不是一个binlog event的名称，mysql的binlog event中也没有一个叫做`BEGIN`的event，这里只不过是工具`mysqlbinlog`展示的时候，展示出来的名称而已。

`BEGIN`其实就是一个QUERY_EVENT。

再接下来还是一个 QUERY_EVENT，对应的就是，`replace into SysDB.StatusTableForHb set tid='7f8dee7fc700',ts = from_unixtime(1593937636),ip='10.112.84.227',port=4069`。

最后是一个`XID_EVENT`，表示事务的结束。你用mysql命令行登录数据库，最后COMMIT的时候，对应的就是这个XID_EVENT，表示事务的提交或者结束。

从上面我们还看到 `sequence_number`和`last_committed`这两个东西，它们是啥含义呢？

- sequence_number
  在每个binlog产生时，从1开始递增，每增加一个事务，sequence_number加1

- last_committed
  用来标识组提交。同一个组提交里多个事务，gtid不同，但lastcommitted确是一致。MySQL正是依据各个事务的lastcommitted来判断它们在不在一个组里;一个组里的lastcommitted与上一个组提交事务的sequencenumber相同。比如，

  ```
  ...... 
  xxxxxxxxxxxx   GTID    last_committed=3        sequence_number=8   
  xxxxxxxxxxxx   GTID    last_committed=3        sequence_number=9   
  xxxxxxxxxxxx   GTID    last_committed=9        sequence_number=10   
  ...... 
  xxxxxxxxxxxx   GTID    last_committed=9        sequence_number=24   
  xxxxxxxxxxxx   GTID    last_committed=24        sequence_number=25   
  ...... 
  ```

  这代表sequencenumber=10到sequencenumber=24的事务在同一个组里(因为lastcommitted都相同，是9)