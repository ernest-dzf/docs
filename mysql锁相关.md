# mysql锁相关 #
## information_schema中Innodb相关表 ##

1. INNODB_TRX 表

		SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX;
	
	- TRX_ID
		
		InnoDB存储引擎内部唯一的事务ID
	- TRX_WEIGHT
		
		事务的权重，反映了一个事务修改和锁住的行数。当发生死锁回滚的时候，优先选择该值最小的进行回滚
		
	- TRX_STATE

		当前事务的状态： RUNNING, LOCK WAIT, ROLLING BACK or COMMITTING
	- TRX_STARTED

		事务的开始时间
	- TRX_REQUESTED_LOCK_ID

		事务等待的锁的ID（如果事务状态不是LOCK WAIT，这个字段是NULL），详细的锁的信息可以连查**INNODB_LOCKS**表
	- TRX_WAIT_STARTED

		事务等待开始的时间 （如果事务状态不是LOCK WAIT，这个字段是NULL）
	- TRX_MYSQL_THREAD_ID

		Mysql中的线程ID，show processlist显示的结果
	- TRX_QUERY

		事务运行的sql语句
	- TRX_OPERATION_STATE

		事务当前操作状态 如updating or deleting，starting index read等
	- TRX_TABLES_IN_USE

		用到的表的数量
	- TRX_TABLES_LOCKED

		加行锁的表的数量
	- TRX_LOCK_STRUCTS
	- TRX_LOCK_MEMORY_BYTES
	- TRX_ROWS_LOCKED

		事务锁住的行数（不是准确数字）,包括了delete-marked的行
	- TRX_ROWS_MODIFIED

		事务插入或者修改的行数
	- TRX_CONCURRENCY_TICKETS
	- TRX_ISOLATION_LEVEL

		隔离级别
	- TRX_UNIQUE_CHECKS
	- TRX_FOREIGN_KEY_CHECKS

		当前事务外键检测，是否开启
	- TRX_LAST_FOREIGN_KEY_ERROR
	- TRX_ADAPTIVE_HASH_LATCHED
	- TRX_ADAPTIVE_HASH_TIMEOUT
2. INNODB_LOCKS 表
3. INNODB_LOCK_WAITS 表