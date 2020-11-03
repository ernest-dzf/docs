# 使用 sysbench 压测mysql

## 安装

1. ```shell
   yum -y install sysbench
   ```

2. 查看安装的版本`sysbench --version`
3. 查看已经安装的包的信息，我们在使用 sysbench 去做压测的时候，会用到包自带的lua文件，这时候就需要知道包的安装路径。`rpm -ql sysbench `

## 使用

需要提前将database建好。

```shell
sysbench /usr/share/sysbench/oltp_insert.lua  --mysql-host=XXX.XXX.XXX.XXX  --mysql-port=3306 --mysql-user=testsbuser  --mysql-password='textpwd' --mysql-db=tssysbench --db-driver=mysql  --tables=15  --table-size=500000  --report-interval=10 --threads=12   --time=120 prepare

```

示例，

```
[root@VM_0_15_centos ~]# pt-online-schema-change --charset=utf8 --no-version-check --user=victor --password=199233 --host=172.19.0.15 P=3306,D=victor,t=sbtest1 --alter="ADD COLUMN column1 tinyint(4) DEFAULT NULL" --execute
No slaves found.  See --recursion-method if host VM_0_15_centos has slaves.
Not checking slave lag because no slaves were found and --check-slave-lag was not specified.
Operation, tries, wait:
  analyze_table, 10, 1
  copy_rows, 10, 0.25
  create_triggers, 10, 1
  drop_triggers, 10, 1
  swap_tables, 10, 1
  update_foreign_keys, 10, 1
Altering `victor`.`sbtest1`...
Creating new table...
Created new table victor._sbtest1_new OK.
Altering new table...
Altered `victor`.`_sbtest1_new` OK.
2020-10-20T21:22:44 Creating triggers...
2020-10-20T21:22:44 Created triggers OK.
2020-10-20T21:22:44 Copying approximately 500 rows...
2020-10-20T21:22:44 Copied rows OK.
2020-10-20T21:22:44 Analyzing new table...
2020-10-20T21:22:44 Swapping tables...
2020-10-20T21:22:44 Swapped original and new tables OK.
2020-10-20T21:22:44 Dropping old table...
2020-10-20T21:22:44 Dropped old table `victor`.`_sbtest1_old` OK.
2020-10-20T21:22:44 Dropping triggers...
2020-10-20T21:22:44 Dropped triggers OK.
Successfully altered `victor`.`sbtest1`.
[root@VM_0_15_centos ~]# 

```

