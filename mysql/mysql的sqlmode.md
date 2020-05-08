# sql_mode

## NO_AUTO_VALUE_ON_ZERO

如果想让自增列在只插入null的时候产生自增序列，就要提前设置mysql的sql_mode包括NO_AUTO_VALUE_ON_ZERO。

如果你不设置的话，那么你对自增列插入0值的时候，也是会产生自增序列的。