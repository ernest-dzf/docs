# mysql外键

## what is 外键

A表的主键，在B表中字段出现，就是外键。



## what is 约束

约束是一种限制，它通过对表的数据做出限制，来确保表的数据的完整性、唯一性。

### 六大约束

- NOT NULL ：非空，用于保证该字段的值不能为空。例如学生表的学生姓名及学号等等

- DEFAULT：默认值，用于保证该字段有默认值。例如学生表的学生性别
- PRIMARY KEY：主键，用于保证该字段的值具有唯一性并且非空。例如学生表的学生学号等
- UNIQUE：唯一，用于保证该字段的值具有唯一性，可以为空。例如注册用户的手机号，身份证号等
- FOREIGN KEY：外键，用于限制两个表的关系，用于保证该字段的值必须来自于主表的关联列的值，在从表添加外键约束，用于引用主表中某些的值



## 使用外键条件

1. 目前只有innodb 支持外键
2. 两张表没有临时表
3. 建立外键关系的对应列必须具有相似的InnoDB内部数据类型
4. 建立外键关系的对应列必须建立了索引
5. 假如显式地给出了CONSTRAINT symbol，那symbol在数据库中必须是唯一的。假如没有显式地给出，InnoDB会自动地创建
6. 外键不一定是另外一个表（主表）的主键，但必须是唯一性索引。

如果**子表**试图创建一个在**父表中不存在**的外键值，InnoDB会**拒绝任何**INSERT或UPDATE操作。



如果**父表**试图UPDATE或者DELETE任何**子表**中存在或匹配的外键值，最终动作取决于**外键约束定义**中的ON UPDATE和ON DELETE选项。

InnoDB支持5种不同的动作，如果没有指定ON DELETE或者ON UPDATE，**默认的动作为RESTRICT**：

- **CASCADE**， 从父表中删除或更新对应的行，同时自动地删除或更新子表中匹配的行。ON DELETE CANSCADE和ON UPDATE CANSCADE都被InnoDB所支持。
- SET NULL，从父表中删除或更新对应的行，同时将子表中的外键列设为空。注意，在外键列没有被设为NOT NULL时才有效。
- NO ACTION，和 RESTRICT 一样。
- RESTRICT，拒绝删除或者更新父表。指定RESTRICT（或者NO ACTION）和忽略ON DELETE或者ON UPDATE选项的效果是一样的。就是说默认就是RESTRICT。



我们在创建外键约束定义的时候，可以指定ON UPDATE、ON DELETE动作，

```mysql
alter table person add CONSTRAINT fk_id foreign key (dept_id) REFERENCES dept(did) ON UPDATE CASCADE;
```



## 外键范例

比如`dept`和`person`这两张表。

```mysql
CREATE TABLE `dept` (
  `did` int(11) NOT NULL AUTO_INCREMENT,
  `dname` varchar(50) NOT NULL COMMENT 'department name',
  PRIMARY KEY (`did`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8

CREATE TABLE `person` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(50) NOT NULL,
  `age` tinyint(4) DEFAULT '0',
  `sex` enum('male','female') NOT NULL DEFAULT 'male',
  `dept_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `fk_did` (`dept_id`),
  CONSTRAINT `fk_did` FOREIGN KEY (`dept_id`) REFERENCES `dept` (`did`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8

```

其中 `dept`表被称为主表，`person`表被称为从表。

比如上面，如果想在`person`表中插入一条记录，那么这条记录中的 `dept_id`字段，其取值必须在主表`dept`中存在。

如果不存在，那么就会报错，

```mysql
MySQL [test]> select * from dept;
+-----+---------+
| did | dname   |
+-----+---------+
|   1 | english |
+-----+---------+
1 row in set (0.00 sec)

MySQL [test]> insert into `person` values (2, 'victor', 12, 'male', 2);
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`test`.`person`, CONSTRAINT `fk_did` FOREIGN KEY (`dept_id`) REFERENCES `dept` (`did`))
MySQL [test]> 

```

有时候，我们为了能正常将表数据导入到这两张表，回临时将 **外键约束** 关掉，然后再打开。



```mysql
MySQL [test]> SET FOREIGN_KEY_CHECKS=0;
Query OK, 0 rows affected (0.00 sec)

MySQL [test]> insert into `person` values (2, 'victor', 12, 'male', 2);
Query OK, 1 row affected (0.00 sec)

MySQL [test]> SET FOREIGN_KEY_CHECKS=1;
Query OK, 0 rows affected (0.00 sec)

```



## 添加外键

如果表已经建好了，如何添加外键呢？

```mysql
alter table person add CONSTRAINT fk_id foreign key (dept_id) REFERENCES dept(did);
```

```mysql
alter table person add CONSTRAINT fk_id foreign key (dept_id) REFERENCES dept(did) ON UPDATE CASCADE;
```

