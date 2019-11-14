---
title           : read committed与statement-based replication
date            : 2017-09-15
tags            : ["技术研究", "MySQL"]
keywords        : ["read committed", "replication"]
category        : "研发"
isCJKLanguage   : true
---

# 疑问

在MySQL的[Transaction Isolation Level] (https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html)文档里读到这么一句话：

> If you use READ COMMITTED, you must use row-based binary logging.

这引发了我的好奇心：为什么呢？READ COMMITTED和statement-based replication会引发数据不一致吗？为什么会不一致呢？

# 猜测

根据READ COMMITTED的特征，我的第一猜测是可能是因为multi-threaded replication。但稍微一细想就否了：

- MySQL v5.6里，只有不同的table才可以并行replay。
- MySQL v5.7里，增加了logical clock，但并不影响。
- MySQL v8.0里，强化了multi-threaded replication，但只有workset完全不重叠的transaction才会并行。

因此，multi-threaded应该不是问题的关键。

# 分析

自己琢磨了一会没想明白。搜的资料全是讲如何解决（改用row-based），没有讲为什么。

又花了些功夫在MySQL Bugs Home里找到一些资料。顺着其中的思路弄明白了缘由：确实和multi-threaded无关。而是由MySQL记录transaction的机制决定的。

MySQL在commit时才将transaction一次性写入binary log。所以：

- 在slave上，transaction是一次性replay的，在`START TRANSACTION`和`COMMIT`之间的所有操作都针对的是同一个数据集。
- 在master上，transaction的`START TRANSACTION`和`COMMIT`是可能有时间跨度的，由于READ COMMITTED的特性，这个时间跨度中操作的数据集可能是不同的。

因此，replication后，master和slave上的数据可能会不一致。用SQL语句模拟一次可能会更好理解。

# 模拟

```sql
CREATE TABLE `t` (
  `a` int(11) DEFAULT NULL,
  `b` int(11) DEFAULT NULL,
  KEY `a` (`a`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

INSERT INTO t VALUES(10,2),(20,1);

# trx-A
(A) SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
(A) SET autocommit=0;
(A) UPDATE t SET a=11 where b=2;    # 此时master数据：(11,2),(20,1)

# trx-B
(B) SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
(B) SET autocommit=0;
(B) UPDATE t SET b=2 where b=1;     # 此时master数据：(10,2),(20,2)。
(B) COMMIT;                         # slave此时执行B，数据变为：(10,2),(20,2)。

# trx-A
(A) COMMIT;                         # 此时master数据：(11,2),(20,2)。
                                    # 记录到master的bin log、replicate到slave。slave此时执行A，数据变为：(11,2),(11,2)
```


执行完毕后，master和slave数据就不一致了：

- master: (11,2),(20,2)
- slave:  (11,2),(11,2)


# 资料

1. [MySQL :: MySQL 5.7 Reference Manual :: 14.5.2.1 Transaction Isolation Levels](https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html)
2. [MySQL :: MySQL 5.7 Reference Manual :: 14.5.1 InnoDB Locking](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html)
3. [MySQL Bugs: #23051: READ COMMITTED breaks mixed and statement-based replication](https://bugs.mysql.com/bug.php?id=23051)
4. [MySQL :: MySQL 5.7 Reference Manual :: 13.3.1 START TRANSACTION, COMMIT, and ROLLBACK Syntax](https://dev.mysql.com/doc/refman/5.7/en/commit.html)
5. [What’s New With MySQL Replication in MySQL 8.0](https://severalnines.com/blog/what-s-new-mysql-replication-mysql-80)
