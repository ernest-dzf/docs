# iotop

```shell
Total DISK READ :       0.00 B/s | Total DISK WRITE :      40.92 M/s
Actual DISK READ:       0.00 B/s | Actual DISK WRITE:     192.05 K/s
  TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND
  525 be/4 user_00     0.00 B/s    3.00 K/s  0.00 %  0.00 % ./router_update /data/tdsql_run/4161/gateway/conf/instance_24161.cnf
  536 be/4 user_00     0.00 B/s    6.00 K/s  0.00 %  0.00 % ./mysqlreport ../conf/mysqlagent_4200.xml
  709 be/4 user_00     0.00 B/s    3.00 K/s  0.00 %  0.00 % ./mysqlreport ../conf/mysqlagent_4201.xml
  971 be/4 user_00     0.00 B/s    3.00 K/s  0.00 %  0.00 % ./router_update /data/tdsql_run/4198/gateway/conf/instance_24198.cnf
 1052 be/4 user_00     0.00 B/s    3.00 K/s  0.00 %  0.00 % ./router_update /data/tdsql_run/4209/gateway/conf/instance_24209.cnf
 1458 be/4 user_00     0.00 B/s   12.00 K/s  0.00 %  0.00 % ./mysqlreport ../conf/mysqlagent_4202.xml
 1736 be/4 user_00     0.00 B/s   12.00 K/s  0.00 %  0.00 % ./mysqlreport ../conf/mysqlagent_4203.xml
 2332 be/4 user_00     0.00 B/s    3.00 K/s  0.00 %  0.00 % ./router_update /data/tdsql_run/4211/gateway/conf/instance_24211.cnf
 2825 be/4 user_00     0.00 B/s    3.00 K/s  0.00 %  0.00 % ./router_update /data/tdsql_run/4179/gateway/conf/instance_24179.cnf
 2834 be/4 user_00     0.00 B/s    3.00 K/s  0.00 %  0.00 % ./mysqlreport ../conf/mysqlagent_4206.xml
 2943 be/4 user_00     0.00 B/s    3.00 K/s  0.00 %  0.00 % ./mysqlreport ../conf/mysqlagent_4205.xml
 3171 be/4 user_00     0.00 B/s    6.00 K/s  0.00 %  0.00 % ./mysqlreport ../conf/mysqlagent_4207.xml
```

**iotop**可以以线程的维度展示io使用情况。

其中 `IO`列表示，某个采样周期内，该线程耗费在IO上的时间占比。

也就是说，如果 `IO`列取值为 99%的话，那么100秒就有99秒都耗费在IO上了，IO是瓶颈。

关于 **iotop**的进一步解释可以看这篇[文章](https://unix.stackexchange.com/questions/248197/iotop-showing-1-5-mb-s-of-disk-write-but-all-programs-have-0-00-b-s/248218#248218)。

