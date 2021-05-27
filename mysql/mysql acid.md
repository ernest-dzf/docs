# MySQL ACID

经常说到mysql的acid（原子性、一致性、隔离线和持久性）。

这里**一致性**如何理解呢？

事务的一致性和数据库预先定义的约束有关，满足了约束即满足了一致性。

> Consistency ensures that a transaction can only bring the database from one valid state to another, maintaining database invariants: any data written to the database must be valid according to all defined rules, including constraints, cascades,triggers, and any combination thereof. This prevents database corruption by an illegal transaction, but does not guarantee that a transaction is correct.

数据库事务的一致性是指：保证事务只能把数据库从一个有效（正确）的状态“转移”到另一个有效（正确）的状态。

那么，什么是数据库的有效(正确）的状态？

满足给这个数据库pre-defined的一些规则都是 valid 的。这些规则有哪些呢，比如说constraints, cascades,triggers 及它们的组合等。具体到某个表的某个字段，比如你在定义表的时候，给这个字段的类型是number类型，并且它的值不能小于0，那么你在某个 transaction 中给这个字段插入（更改）为一个 String 值或者是负值是不可以的，这不是一个“合法”的transaction，也就是说它不满足我们给数据库定义的一些规则（约束条件）。

> This prevents database corruption by an illegal transaction, but does not guarantee that a transaction is correct.

这又怎么理解呢？在数据库的角度来看，它只关心 transaction 符不符合定义好的规则，符合的就是legal的，不符合的就是illegal的。transaction 是否正确是从应用层的角度来看的，数据库并不知道你应用层的逻辑意义，它不保证应用层的transaction的正确性，这个逻辑正确性是由应用层的programmer来保证的。