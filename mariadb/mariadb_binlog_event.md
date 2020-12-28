# binlog event 介绍

## Fake ROTATE_EVENT

当一个slave连上一个mariadb 10 的master时，收到的第一个binlog event就是 `Fake ROTATE_EVENT`。

这个event是伪造的，实际上你在master服务器的binlog文件中找不到对应的event。

它存在的目的是为了告诉slave，当前master的binlog 文件的文件名。

当我们使用gtid构造复制关系的时候，这个毋庸置疑是有用的，因为slave在挂上master时，是没有去指定binlog的filename的（传统的基于file+position是必须给出的）。slave可以通过`Fake ROTATE_EVENT`获知对应的master上的binlog file name。

>**Note**: the fake ROTATE_EVENT event is not written in the binlog file.
>It's created by the master and sent to new connected slave before [FORMAT_DESCRIPTION_EVENT](https://mariadb.com/kb/en/format_description_event/)

Event Type 取值为 `ROTATE_EVENT (0x4)`。

怎么和 `ROTATE_EVEN`区分的呢？通过 Event Header的特征来判断。比如

- Timestamp set to 0
- Next Pos is set to 0
- Flags are set to LOG_ARTIFICIAL_F (0x20)
- Event Tye is ROTATE_EVENT

参考链接：https://mariadb.com/kb/en/fake-rotate_event/



## GTID_EVENT

用于启动一个新的事务。

> used to start a new transaction event group, instead of the old BEGIN query event, and also to mark stand-alone (ddl)

举个例子，

```mysql
begin;
insert into tb1 values (1);
commit;
```

类似上面这个简单的事务，在binlog中，首先会有1个`GTID_EVENT`，标识一个新的事务了，里面包含有gtid。

这里和mysql不一样的地方是，mysql开启一个新的事务，对应的event是一个`QUERY_EVENT`，通过判断`QUERY_EVENT`里面的内容（等于`BEGIN`），来判断是否是开启一个新事务。

而mariadb不是，mariadb则通过`GTID_EVENT`来判断开启一个新事务。



### Event Header

- Type[1] = 0xa2

- Flags[2] = 08 00 => LOG_EVENT_SUPPRESS_USE_F

其中`LOG_EVENT_SUPPRESS_USE_F`表示，在一个语句被记录前，抑制产生USE语句。用于任何不需要使用默认数据库的事件中，比如`CREATE DATABASE`和`DROP DATABASE`。你创建数据库的时候，不需要再多此一举地`USE xxxx`，之后再去创建吧。

Flags有2个字节，第二个字节有一些bit控制位，含义如下：

| FL_STANDALONE      | 1    | Set when there is no terminating COMMIT event.               |
| ------------------ | ---- | ------------------------------------------------------------ |
| FL_GROUP_COMMIT_ID | 2    | Set when event group is part of a group commit on the master. Groups with same commit_id are part of the same group commit. |
| FL_TRANSACTIONAL   | 4    | Set for an event group that can be safely rolled back (no MyISAM, eg.). |
| FL_ALLOW_PARALLEL  | 8    | Reflects the (negation of the) value of @@SESSION.skip_parallel_replication at the time of commit. |
| FL_WAITED          | 16   | Set if a row lock wait (or other wait) is detected during the execution of the transaction. |
| FL_DDL             | 32   | Set for event group containing DDL.                          |

以`FL_STANDALONE`为例，表示这个GTID_EVENT没有对应的COMMIT EVENT。什么场景下会这样呢？

你做DDL操作的时候，比如，`alter table tb1 modify column id int(20)`，记录到binlog中的会有2个Event，分别是

GTID_EVENT和QUERY_EVENT。其中GTID_EVENT的Flags只有FL_STANDALONE这个标志位被置为1，表示没有对应的COMMIT EVENT（`XID_EVENT`）。

```
=== MariadbGTIDEvent ===
Date: 2020-07-05 14:20:33
Log position: 5186164
Event size: 38
GTID: {0 301117090 466218}
Flags: 1
CommitID: 0

=== QueryEvent ===
Date: 2020-07-05 14:20:33
Log position: 5186271
Event size: 107
Slave proxy ID: 675297
Execution time: 0
Error code: 0
Schema: victor
Query: alter table tb1 modify column `id` int(20)
GTIDSet: 0-301117090-466218

```

GTID_EVENT 之后就紧跟着QUERY_EVENT，表示做的DDL操作。

我们注意到后面是没有COMMIT EVENT（`XID_EVENT`）跟着的。

## XID_EVENT

> An XID event is generated for a COMMIT of a transaction that modifies one or more tables of an XA-capable storage engine.

如果你使用的mariadb 存储引擎是 XA-capable，那么每次提交事务的时候（就是`COMMIT`），都会在最后面添加一个`XID_EVENT`。XA-capable，现在一般都是XA-capable了。

你可以认为它就是你使用`mysqlbinlog`分析binlog文件时，看到的COMMIT语句，比如下面

```
#200705 14:23:23 server id 301117090  end_log_pos 5243348       Xid = 8539603
COMMIT/*!*/;
# at 5243348

```

## ROTATE_EVENT

当binlog文件超过配置的大小时，会有1个`ROTATE_EVENT`被写入到binlog文件的末尾，这个event会有下一个binlog文件的信息。

`ROTATE_EVENT`是在master上真实产生的，这点和之前介绍的`Fake ROTATE_EVENT`不一样。

如果你在master上使用`FLUSH LOGS`，也会产生一个`ROTATE_EVENT`。就是切分binlog文件。

Event Type 取值为 `ROTATE_EVENT (0x4)`。

### HEADER

- The Event Type is set ROTATE_EVENT (0x4)

### Fields

- [uint<8>](https://mariadb.com/kb/en/protocol-data-types/#fixed-length-bytes) ，下一个binlog文件的第一个event的位移，目前默认都是4
- [string<EOF>](https://mariadb.com/kb/en/protocol-data-types/#end-of-file-length-strings)，下一个binlog文件名，注意这个文件名没有以`\0`字符结尾。

```
T 127.0.0.1:8808 -> 127.0.0.1:57157 [AP]
  30 00 00 4d 00 bc 4e 21    5a 04 d9 27 00 00 2f 00    0..M..N!Z..'../.
  00 00 c0 01 00 00 00 00    04 00 00 00 00 00 00 00    ................
  6d 79 73 71 6c 2d 62 69    6e 2e 30 30 30 30 31 39    mysql-bin.000019
  b2 bc db bf                                           ....    
```

