# mysqldump使用#

## 命令 ##

`mysqldump -h主机名 -P端口 -u用户名 -p密码 --databases 数据库名 > 文件名.sql`


## --master-data ##

## --dump-slave ##

## --lock-all-tables ##

官方解释，

>Lock all tables across all databases. This is achieved by acquiring a global read lock for the duration of the whole dump. This option automatically turns off `--single-transaction` and `--lock-tables`


就是说在整个的dump期间，会加一个全局read lock（FTWRL）。这样在整个dump过程到`unlock tables`阶段，其他线程不可写入，从而保证数据的一致性。

添加这个选项会自动将`--single-transaction` 和 `--lock-tables`选项关掉。

## --single-transaction ##

对于myisam表，这个选项其实是没有意义的。

myisam不支持事务，在整个dump过程中无法保证可重复读，无法得到一致性的备份。

而innodb表在备份过程中，虽然其他线程可能也在写数据，但是dump出来的数据能保证是备份开始时那个binlog pos的数据。

## --lock-tables ##