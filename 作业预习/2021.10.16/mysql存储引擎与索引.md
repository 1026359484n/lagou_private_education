# MySQL 基础

## 存储引擎

### 存储引擎相关的命令

**查看 MySQL 提供的所有存储引擎**

```
mysql> show engines;
```

[![查看MySQL提供的所有存储引擎](https://camo.githubusercontent.com/aea99555c365862b0ed838affdc1925feaade72fa1c75aa3a6e21c8181cb7f7a/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f6d7973716c2d656e67696e65732e706e67)](https://camo.githubusercontent.com/aea99555c365862b0ed838affdc1925feaade72fa1c75aa3a6e21c8181cb7f7a/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f6d7973716c2d656e67696e65732e706e67)

从上图我们可以查看出 MySQL 当前默认的存储引擎是 InnoDB，并且在 5.7 版本所有的存储引擎中只有 InnoDB 是事务性存储引擎，也就是说只有 InnoDB 支持事务。

**查看 MySQL 当前默认的存储引擎**

我们也可以通过下面的命令查看默认的存储引擎。

```
mysql> show variables like '%storage_engine%';
```

**查看表的存储引擎**

```
show table status like "table_name" ;
```

[![查看表的存储引擎](https://camo.githubusercontent.com/4de1f7ecddcf627a6a3d2401aadf56b3febee49858a24d72f376a26db895c9ea/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f2545362539462541352545372539432538422545382541312541382545372539412538342545352541442539382545352538322541382545352542432539352545362539332538452e706e67)](https://camo.githubusercontent.com/4de1f7ecddcf627a6a3d2401aadf56b3febee49858a24d72f376a26db895c9ea/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f2545362539462541352545372539432538422545382541312541382545372539412538342545352541442539382545352538322541382545352542432539352545362539332538452e706e67)

### MyISAM 和 InnoDB 的区别

MySQL 5.5 之前，MyISAM 引擎是 MySQL 的默认存储引擎，可谓是风光一时。

虽然，MyISAM 的性能还行，各种特性也还不错（比如全文索引、压缩、空间函数等）。但是，MyISAM 不支持事务和行级锁，而且最大的缺陷就是崩溃后无法安全恢复。

5.5 版本之后，MySQL 引入了 InnoDB（事务性数据库引擎），MySQL 5.5 版本后默认的存储引擎为 InnoDB。小伙子，一定要记好这个 InnoDB ，你每次使用 MySQL 数据库都是用的这个存储引擎吧？

言归正传！咱们下面还是来简单对比一下两者：

InnoDB和MyISAM是使用MySQL时最常用的两种引擎类型，我们重点来看下两者区别。

**事务和外键**

InnoDB支持事务和外键，具有安全性和完整性，适合大量insert或update操作

MyISAM不支持事务和外键，它提供高速存储和检索，适合大量的select查询操作

**锁机制**

InnoDB支持行级锁，锁定指定记录。基于索引来加锁实现。

MyISAM支持表级锁，锁定整张表。

**索引结构**

InnoDB使用聚集索引（聚簇索引），索引和记录在一起存储，既缓存索引，也缓存记录。

MyISAM使用非聚集索引（非聚簇索引），索引和记录分开。

**并发处理能力**

MyISAM使用表锁，会导致写操作并发率低，读之间并不阻塞，读写阻塞。

InnoDB读写阻塞可以与隔离级别有关，可以采用多版本并发控制（MVCC）来支持高并发

**存储文件**

InnoDB表对应两个文件，一个.frm表结构文件，一个.ibd数据文件。InnoDB表最大支持64TB；

MyISAM表对应三个文件，一个.frm表结构文件，一个MYD表数据文件，一个.MYI索引文件。从

MySQL5.0开始默认限制是256TB。

![image-20211016172703039](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.10.16/image-20211016172703039-4376425.png)

**数据库异常崩溃后的安全恢复**

MyISAM 不支持，而 InnoDB 支持。

使用 InnoDB 的数据库在异常崩溃后，数据库重新启动的时候会保证数据库恢复到崩溃前的状态。这个恢复的过程依赖于 `redo log` 。

🌈 拓展一下：

- MySQL InnoDB 引擎使用 **redo log(重做日志)** 保证事务的**持久性**，使用 **undo log(回滚日志)** 来保证事务的**原子性**。
- MySQL InnoDB 引擎通过 **锁机制**、**MVCC** 等手段来保证事务的隔离性（ 默认支持的隔离级别是 **`REPEATABLE-READ`** ）。
- 保证了事务的持久性、原子性、隔离性之后，一致性才能得到保障。

### 关于 MyISAM 和 InnoDB 的选择问题

适用场景

**MyISAM**

​	不需要事务支持（不支持）

​	并发相对较低（锁定机制问题）

​	数据修改相对较少，以读为主

​	数据一致性要求不高

**InnoDB**

​	需要事务支持（具有较好的事务特性）

​	行级锁定对高并发有很好的适应能力

​	数据更新较为频繁的场景

​	数据一致性要求较高

​	硬件设备内存较大，可以利用InnoDB较好的缓存能力来提高内存利用率，减少磁盘IO

总结

​	两种引擎该如何选择？

​	是否需要事务？有，InnoDB

​	是否存在并发修改？有，InnoDB

​	是否追求快速查询，且数据修改少？是，MyISAM

​	在绝大多数情况下，推荐使用InnoDB

《MySQL 高性能》上面有一句话这样写到:

> 不要轻易相信“MyISAM 比 InnoDB 快”之类的经验之谈，这个结论往往不是绝对的。在很多我们已知场景中，InnoDB 的速度都可以让 MyISAM 望尘莫及，尤其是用到了聚簇索引，或者需要访问的数据都可以放入内存的应用。

对于咱们日常开发的业务系统来说，已经几乎找不到什么理由再使用 MyISAM 作为自己的 MySQL 数据库的存储引擎。

## 锁机制与 InnoDB 锁算法

**MyISAM 和 InnoDB 存储引擎使用的锁：**

- MyISAM 采用表级锁(table-level locking)。
- InnoDB 支持行级锁(row-level locking)和表级锁,默认为行级锁

**表级锁和行级锁对比：**

- **表级锁：** MySQL 中锁定 **粒度最大** 的一种锁，对当前操作的整张表加锁，实现简单，资源消耗也比较少，加锁快，不会出现死锁。其锁定粒度最大，触发锁冲突的概率最高，并发度最低，MyISAM 和 InnoDB 引擎都支持表级锁。
- **行级锁：** MySQL 中锁定 **粒度最小** 的一种锁，只针对当前操作的行进行加锁。 行级锁能大大减少数据库操作的冲突。其加锁粒度最小，并发度高，但加锁的开销也最大，加锁慢，会出现死锁。

**InnoDB 存储引擎的锁的算法有三种：**

- Record lock：记录锁，单个行记录上的锁
- Gap lock：间隙锁，锁定一个范围，不包括记录本身
- Next-key lock：record+gap 临键锁，锁定一个范围，包含记录本身

## 查询缓存

执行查询语句的时候，会先查询缓存。不过，MySQL 8.0 版本后移除，因为这个功能不太实用

`my.cnf` 加入以下配置，重启 MySQL 开启查询缓存

```
query_cache_type=1
query_cache_size=600000
```

MySQL 执行以下命令也可以开启查询缓存

```
set global  query_cache_type=1;
set global  query_cache_size=600000;
```

如上，**开启查询缓存后在同样的查询条件以及数据情况下，会直接在缓存中返回结果**。这里的查询条件包括查询本身、当前要查询的数据库、客户端协议版本号等一些可能影响结果的信息。（**查询缓存不命中的情况：（1）**）因此任何两个查询在任何字符上的不同都会导致缓存不命中。此外，（**查询缓存不命中的情况：（2）**）如果查询中包含任何用户自定义函数、存储函数、用户变量、临时表、MySQL 库中的系统表，其查询结果也不会被缓存。

（**查询缓存不命中的情况：（3）**）**缓存建立之后**，MySQL 的查询缓存系统会跟踪查询中涉及的每张表，如果这些表（数据或结构）发生变化，那么和这张表相关的所有缓存数据都将失效。

**缓存虽然能够提升数据库的查询性能，但是缓存同时也带来了额外的开销，每次查询后都要做一次缓存操作，失效后还要销毁。** 因此，开启查询缓存要谨慎，尤其对于写密集的应用来说更是如此。如果开启，要注意合理控制缓存空间大小，一般来说其大小设置为几十 MB 比较合适。此外，**还可以通过 sql_cache 和 sql_no_cache 来控制某个查询语句是否需要缓存：**

```
select sql_no_cache count(*) from usr;
```

## 事务

### 何为事务？

一言蔽之，**事务是逻辑上的一组操作，要么都执行，要么都不执行。**

**可以简单举一个例子不？**

事务最经典也经常被拿出来说例子就是转账了。假如小明要给小红转账 1000 元，这个转账会涉及到两个关键操作就是：

1. 将小明的余额减少 1000 元
2. 将小红的余额增加 1000 元。

事务会把这两个操作就可以看成逻辑上的一个整体，这个整体包含的操作要么都成功，要么都要失败。

这样就不会出现小明余额减少而小红的余额却并没有增加的情况。

### 何为数据库事务？

数据库事务在我们日常开发中接触的最多了。如果你的项目属于单体架构的话，你接触到的往往就是数据库事务了。

平时，我们在谈论事务的时候，如果没有特指**分布式事务**，往往指的就是**数据库事务**。

**那数据库事务有什么作用呢？**

简单来说：数据库事务可以保证多个对数据库的操作（也就是 SQL 语句）构成一个逻辑上的整体。构成这个逻辑上的整体的这些数据库操作遵循：**要么全部执行成功,要么全部不执行** 。

```
# 开启一个事务
START TRANSACTION;
# 多条 SQL 语句
SQL1,SQL2...
## 提交事务
COMMIT;
```

[![img](https://camo.githubusercontent.com/d9c8448f21fb27f4565e846dc1b63077c01f3e27e03764940f4e1b8ec4731514/68747470733a2f2f67756964652d626c6f672d696d616765732e6f73732d636e2d7368656e7a68656e2e616c6979756e63732e636f6d2f323032302d31322f3634302d32303230313230373136303535343637372e706e67)](https://camo.githubusercontent.com/d9c8448f21fb27f4565e846dc1b63077c01f3e27e03764940f4e1b8ec4731514/68747470733a2f2f67756964652d626c6f672d696d616765732e6f73732d636e2d7368656e7a68656e2e616c6979756e63732e636f6d2f323032302d31322f3634302d32303230313230373136303535343637372e706e67)

另外，关系型数据库（例如：`MySQL`、`SQL Server`、`Oracle` 等）事务都有 **ACID** 特性：

[![事务的特性](https://camo.githubusercontent.com/a56cfc4add00ae0a268516f154d400fecd22a80b1bde794c8188b1293ee093e7/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f2545342542412538422545352538412541312545372538392542392545362538302541372e706e67)](https://camo.githubusercontent.com/a56cfc4add00ae0a268516f154d400fecd22a80b1bde794c8188b1293ee093e7/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f2545342542412538422545352538412541312545372538392542392545362538302541372e706e67)

### 何为 ACID 特性呢？

1. **原子性**（`Atomicity`） ： 事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；
2. **一致性**（`Consistency`）： 执行事务前后，数据保持一致，例如转账业务中，无论事务是否成功，转账者和收款人的总额应该是不变的；
3. **隔离性**（`Isolation`）： 并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的；
4. **持久性**（`Durability`）： 一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。

**数据事务的实现原理呢？**

我们这里以 MySQL 的 InnoDB 引擎为例来简单说一下。

MySQL InnoDB 引擎使用 **redo log(重做日志)** 保证事务的**持久性**，使用 **undo log(回滚日志)** 来保证事务的**原子性**。

MySQL InnoDB 引擎通过 **锁机制**、**MVCC** 等手段来保证事务的隔离性（ 默认支持的隔离级别是 **`REPEATABLE-READ`** ）。

保证了事务的持久性、原子性、隔离性之后，一致性才能得到保障。

### 并发事务带来哪些问题?

在典型的应用程序中，多个事务并发运行，经常会操作相同的数据来完成各自的任务（多个用户对同一数据进行操作）。并发虽然是必须的，但可能会导致以下的问题。

- **脏读（Dirty read）:** 当一个事务正在访问数据并且对数据进行了修改，而这种修改还没有提交到数据库中，这时另外一个事务也访问了这个数据，然后使用了这个数据。因为这个数据是还没有提交的数据，那么另外一个事务读到的这个数据是“脏数据”，依据“脏数据”所做的操作可能是不正确的。
- **丢失修改（Lost to modify）:** 指在一个事务读取一个数据时，另外一个事务也访问了该数据，那么在第一个事务中修改了这个数据后，第二个事务也修改了这个数据。这样第一个事务内的修改结果就被丢失，因此称为丢失修改。 例如：事务 1 读取某表中的数据 A=20，事务 2 也读取 A=20，事务 1 修改 A=A-1，事务 2 也修改 A=A-1，最终结果 A=19，事务 1 的修改被丢失。
- **不可重复读（Unrepeatable read）:** 指在一个事务内多次读同一数据。在这个事务还没有结束时，另一个事务也访问该数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改导致第一个事务两次读取的数据可能不太一样。这就发生了在一个事务内两次读到的数据是不一样的情况，因此称为不可重复读。
- **幻读（Phantom read）:** 幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在的记录，就好像发生了幻觉一样，所以称为幻读。

**不可重复读和幻读区别：**

不可重复读的重点是修改比如多次读取一条记录发现其中某些列的值被修改，幻读的重点在于新增或者删除比如多次读取一条记录发现记录增多或减少了。

### 事务隔离级别有哪些?

SQL 标准定义了四个隔离级别：

- **READ-UNCOMMITTED(读取未提交)：** 最低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**。
- **READ-COMMITTED(读取已提交)：** 允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**。
- **REPEATABLE-READ(可重复读)：** 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生**。
- **SERIALIZABLE(可串行化)：** 最高的隔离级别，完全服从 ACID 的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。

------

| 隔离级别         | 脏读 | 不可重复读 | 幻读 |
| ---------------- | ---- | ---------- | ---- |
| READ-UNCOMMITTED | √    | √          | √    |
| READ-COMMITTED   | ×    | √          | √    |
| REPEATABLE-READ  | ×    | ×          | √    |
| SERIALIZABLE     | ×    | ×          | ×    |

### MySQL 的默认隔离级别是什么?

MySQL InnoDB 存储引擎的默认支持的隔离级别是 **REPEATABLE-READ（可重读）**。我们可以通过`SELECT @@tx_isolation;`命令来查看，MySQL 8.0 该命令改为`SELECT @@transaction_isolation;`

```
mysql> SELECT @@tx_isolation;
+-----------------+
| @@tx_isolation  |
+-----------------+
| REPEATABLE-READ |
+-----------------+
```

~~这里需要注意的是：与 SQL 标准不同的地方在于 InnoDB 存储引擎在 **REPEATABLE-READ（可重读）** 事务隔离级别下使用的是 Next-Key Lock 锁算法，因此可以避免幻读的产生，这与其他数据库系统(如 SQL Server)是不同的。所以说 InnoDB 存储引擎的默认支持的隔离级别是 **REPEATABLE-READ（可重读）** 已经可以完全保证事务的隔离性要求，即达到了 SQL 标准的 **SERIALIZABLE(可串行化)** 隔离级别。~~

🐛 问题更正：**MySQL InnoDB 的 REPEATABLE-READ（可重读）并不保证避免幻读，需要应用使用加锁读来保证。而这个加锁度使用到的机制就是 Next-Key Locks。**

因为隔离级别越低，事务请求的锁越少，所以大部分数据库系统的隔离级别都是 **READ-COMMITTED(读取提交内容)** ，但是你要知道的是 InnoDB 存储引擎默认使用 **REPEATABLE-READ（可重读）** 并不会有任何性能损失。

InnoDB 存储引擎在 **分布式事务** 的情况下一般会用到 **SERIALIZABLE(可串行化)** 隔离级别。

🌈 拓展一下(以下内容摘自《MySQL 技术内幕：InnoDB 存储引擎(第 2 版)》7.7 章)：

> InnoDB 存储引擎提供了对 XA 事务的支持，并通过 XA 事务来支持分布式事务的实现。分布式事务指的是允许多个独立的事务资源（transactional resources）参与到一个全局的事务中。事务资源通常是关系型数据库系统，但也可以是其他类型的资源。全局事务要求在其中的所有参与的事务要么都提交，要么都回滚，这对于事务原有的 ACID 要求又有了提高。另外，在使用分布式事务时，InnoDB 存储引擎的事务隔离级别必须设置为 SERIALIZABLE。



# 索引

## 1. 索引类型

​	索引可以提升查询速度，会影响where查询，以及order by排序。MySQL索引类型如下：

​		从索引存储结构划分：B Tree索引、Hash索引、FULLTEXT全文索引、R Tree索引

​		从应用层次划分：普通索引、唯一索引、主键索引、复合索引

​		从索引键值类型划分：主键索引、辅助索引（二级索引）

​		从数据存储和索引键值逻辑关系划分：聚集索引（聚簇索引）、非聚集索引（非聚簇索引）

### **1.1** **普通索引**

这是最基本的索引类型，基于普通字段建立的索引，没有任何限制。

创建普通索引的方法如下：

CREATE INDEX <索引的名字> ON tablename (字段名);

ALTER TABLE tablename ADD INDEX [索引的名字] (字段名);

CREATE TABLE tablename ( [...], INDEX [索引的名字] (字段名) );

### **1.2** **唯一索引**

与"普通索引"类似，不同的就是：索引字段的值必须唯一，但允许有空值 。在创建或修改表时追加唯一

约束，就会自动创建对应的唯一索引。

创建唯一索引的方法如下：

CREATE UNIQUE INDEX <索引的名字> ON tablename (字段名);

ALTER TABLE tablename ADD UNIQUE INDEX [索引的名字] (字段名);

CREATE TABLE tablename ( [...], UNIQUE [索引的名字] (字段名) ;

### **1.3** **主键索引**

它是一种特殊的唯一索引，不允许有空值。在创建或修改表时追加主键约束即可，每个表只能有一个主

键。

创建主键索引的方法如下：

CREATE TABLE tablename ( [...], PRIMARY KEY (字段名) );

ALTER TABLE tablename ADD PRIMARY KEY (字段名);

### **1.4** **复合索引**

单一索引是指索引列为一列的情况，即新建索引的语句只实施在一列上；用户可以在多个列上建立索

引，这种索引叫做组复合索引（组合索引）。复合索引可以代替多个单一索引，相比多个单一索引复合

索引所需的开销更小。

索引同时有两个概念叫做窄索引和宽索引，窄索引是指索引列为1-2列的索引，宽索引也就是索引列超

过2列的索引，设计索引的一个重要原则就是能用窄索引不用宽索引，因为窄索引往往比组合索引更有

效。

创建组合索引的方法如下：

CREATE INDEX <索引的名字> ON tablename (字段名1，字段名2...);

ALTER TABLE tablename ADD INDEX [索引的名字] (字段名1，字段名2...);CREATE TABLE tablename ( [...], INDEX [索引的名字] (字段名1，字段名2...) );

复合索引使用注意事项：

何时使用复合索引，要根据where条件建索引，注意不要过多使用索引，过多使用会对更新操作效

率有很大影响。

如果表已经建立了(col1，col2)，就没有必要再单独建立（col1）；如果现在有(col1)索引，如果查

询需要col1和col2条件，可以建立(col1,col2)复合索引，对于查询有一定提高。

### **1.5** **全文索引**

查询操作在数据量比较少时，可以使用like模糊查询，但是对于大量的文本数据检索，效率很低。如果

使用全文索引，查询速度会比like快很多倍。在MySQL 5.6 以前的版本，只有MyISAM存储引擎支持全

文索引，从MySQL 5.6开始MyISAM和InnoDB存储引擎均支持。

创建全文索引的方法如下：

CREATE FULLTEXT INDEX <索引的名字> ON tablename (字段名);

ALTER TABLE tablename ADD FULLTEXT [索引的名字] (字段名);

CREATE TABLE tablename ( [...], FULLTEXT KEY [索引的名字] (字段名) ;

和常用的like模糊查询不同，全文索引有自己的语法格式，使用 match 和 against 关键字，比如

select * from user 

where match(name) against('aaa');

全文索引使用注意事项：

全文索引必须在字符串、文本字段上建立。

全文索引字段值必须在最小字符和最大字符之间的才会有效。（innodb：3-84；myisam：4-

84）

全文索引字段值要进行切词处理，按syntax字符进行切割，例如b+aaa，切分成b和aaa

全文索引匹配查询，默认使用的是等值匹配，例如a匹配a，不会匹配ab,ac。如果想匹配可以在布

尔模式下搜索a*

select * from user 

where match(name) against('a*' in boolean mode); 

## 2. 索引原理

MySQL官方对索引定义：是存储引擎用于快速查找记录的一种数据结构。需要额外开辟空间和数据维护

工作。

索引是物理数据页存储，在数据文件中（InnoDB，ibd文件），利用数据页(page)存储。

索引可以加快检索速度，但是同时也会降低增删改操作速度，索引维护需要代价。

索引涉及的理论知识：二分查找法、Hash和B+Tree。

### **2.1** **二分查找法**

二分查找法也叫作折半查找法，它是在有序数组中查找指定数据的搜索算法。它的优点是等值查询、范

围查询性能优秀，缺点是更新数据、新增数据、删除数据维护成本高。

首先定位left和right两个指针计算(left+right)/2

判断除2后索引位置值与目标值的大小比对

索引位置值大于目标值就-1，right移动；如果小于目标值就+1，left移动

举个例子，下面的有序数组有17 个值，查找的目标值是7，过程如下：

第一次查找

![image-20211016175727989](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.10.16/image-20211016175727989.png)

第二次查找

![image-20211016175742533](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.10.16/image-20211016175742533-4378263.png)

第三次查找

![image-20211016175811946](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.10.16/image-20211016175811946.png)

第四次查找

![image-20211016175822649](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.10.16/image-20211016175822649.png)

### **2.2** **Hash结构**

Hash底层实现是由Hash表来实现的，是根据键值 <key,value> 存储数据的结构。非常适合根据key查找

value值，也就是单个key查询，或者说等值查询。其结构如下所示：

![image-20211016175900708](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.10.16/image-20211016175900708.png)

从上面结构可以看出，Hash索引可以方便的提供等值查询，但是对于范围查询就需要全表扫描了。

Hash索引在MySQL 中Hash结构主要应用在Memory原生的Hash索引 、InnoDB 自适应哈希索引。

InnoDB提供的自适应哈希索引功能强大，接下来重点描述下InnoDB 自适应哈希索引。

InnoDB自适应哈希索引是为了提升查询效率，InnoDB存储引擎会监控表上各个索引页的查询，当InnoDB注意到某些索引值访问非常频繁时，会在内存中基于B+Tree索引再创建一个哈希索引，使得内存中的 B+Tree 索引具备哈希索引的功能，即能够快速定值访问频繁访问的索引页。

InnoDB自适应哈希索引：在使用Hash索引访问时，一次性查找就能定位数据，等值查询效率要优于B+Tree。

自适应哈希索引的建立使得InnoDB存储引擎能自动根据索引页访问的频率和模式自动地为某些热点页建立哈希索引来加速访问。另外InnoDB自适应哈希索引的功能，用户只能选择开启或关闭功能，无法进行人工干涉。

```sql
show engine innodb status \G; 

show variables like '%innodb_adaptive%';
```



### **2.3 B+Tree** **结构**

MySQL数据库索引采用的是B+Tree结构，在B-Tree结构上做了优化改造。

**B-Tree结构**

​	索引值和data数据分布在整棵树结构中

​	每个节点可以存放多个索引值及对应的data数据

​	树节点中的多个索引值从左到右升序排列

![image-20211016180157607](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.10.16/image-20211016180157607-4378519.png)

​	B树的搜索：从根节点开始，对节点内的索引值序列采用二分法查找，如果命中就结束查找。没有命中会进入子节点重复查找过程，直到所对应的的节点指针为空，或已经是叶子节点了才结束。

B+Tree结构

​	非叶子节点不存储data数据，只存储索引值，这样便于存储更多的索引值

​	叶子节点包含了所有的索引值和data数据

​	叶子节点用指针连接，提高区间的访问性能

![image-20211016180242154](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.10.16/image-20211016180242154-4378563.png)

相比B树，B+树进行范围查找时，只需要查找定位两个节点的索引值，然后利用叶子节点的指针进行遍历即可。而B树需要遍历范围内所有的节点和数据，显然B+Tree效率高。

### **2.4** **聚簇索引和辅助索引**

聚簇索引和非聚簇索引：B+Tree的叶子节点存放主键索引值和行记录就属于聚簇索引；如果索引值和行记录分开存放就属于非聚簇索引。

主键索引和辅助索引：B+Tree的叶子节点存放的是主键字段值就属于主键索引；如果存放的是非主键值就属于辅助索引（二级索引）。

在InnoDB引擎中，主键索引采用的就是聚簇索引结构存储。

​	聚簇索引（聚集索引）

聚簇索引是一种数据存储方式，InnoDB的聚簇索引就是按照主键顺序构建 B+Tree结构。B+Tree的叶子节点就是行记录，行记录和主键值紧凑地存储在一起。 这也意味着 InnoDB 的主键索引就是数据表本身，它按主键顺序存放了整张表的数据，占用的空间就是整个表数据量的大小。通常说的**主键索引**就是聚集索引。

​	InnoDB的表要求必须要有聚簇索引：

​		如果表定义了主键，则主键索引就是聚簇索引

​		如果表没有定义主键，则第一个非空unique列作为聚簇索引

​		否则InnoDB会从建一个隐藏的row-id作为聚簇索引

​	辅助索引

​		InnoDB辅助索引，也叫作二级索引，是根据索引列构建 B+Tree结构。但在 B+Tree 的叶子节点中只存了索引列和主键的信息。二级索引占用的空间会比聚簇索引小很多， 通常创建辅助索引就是为了提升查询效率。一个表InnoDB只能创建一个聚簇索引，但可以创建多个辅助索引。

![image-20211016180439441](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.10.16/image-20211016180439441.png)

​	非聚簇索引

​		与InnoDB表存储不同，MyISAM数据表的索引文件和数据文件是分开的，被称为非聚簇索引结构。

![image-20211016180506283](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.10.16/image-20211016180506283.png)

## 3. **索引分析与优化**

### **3.1 EXPLAIN**

MySQL 提供了一个 EXPLAIN 命令，它可以对 SELECT 语句进行分析，并输出 SELECT 执行的详细信

息，供开发人员有针对性的优化。例如：

```sql
EXPLAIN SELECT * from user WHERE id < 3;
```

EXPLAIN 命令的输出内容大致如下：

​	![image-20211016180542730](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.10.16/image-20211016180542730-4378746.png)

- **select_type**

  ​	表示查询的类型。常用的值如下：

  ​			SIMPLE ： 表示查询语句不包含子查询或union

  ​			PRIMARY：表示此查询是最外层的查询

  ​			UNION：表示此查询是UNION的第二个或后续的查询DEPENDENT UNION：UNION中的第二个或后续的查询语句，使用了外面查询结果

  ​			UNION RESULT：UNION的结果

  ​			SUBQUERY：SELECT子查询语句

  ​			DEPENDENT SUBQUERY：SELECT子查询语句依赖外层查询的结果。

最常见的查询类型是SIMPLE，表示我们的查询没有子查询也没用到UNION查询。

- **type**

  ​	表示存储引擎查询数据时采用的方式。比较重要的一个属性，通过它可以判断出查询是全表扫描还是基于索引的部分扫描。常用属性值如下，从上至下效率依次增强。

  ​			ALL：表示全表扫描，性能最差。

  ​			index：表示基于索引的全表扫描，先扫描索引再扫描全表数据。

  ​			range：表示使用索引范围查询。使用>、>=、<、<=、in等等。

  ​			ref：表示使用非唯一索引进行单值查询。

  ​			eq_ref：一般情况下出现在多表join查询，表示前面表的每一个记录，都只能匹配后面表的一行结果。

  ​			const：表示使用主键或唯一索引做等值查询，常量查询。

  ​			NULL：表示不用访问表，速度最快。

- **possible_keys**

  ​	表示查询时能够使用到的索引。注意并不一定会真正使用，显示的是索引名称。

- **key**

  ​	表示查询时真正使用到的索引，显示的是索引名称。

- **rows**

  ​	MySQL查询优化器会根据统计信息，估算SQL要查询到结果需要扫描多少行记录。原则上rows是越少效率越高，可以直观的了解到SQL效率高低。

- **key_len**

  ​	表示查询使用了索引的字节数量。可以判断是否全部使用了组合索引。

  ​	key_len的计算规则如下：

  - ​	字符串类型

    ​	字符串长度跟字符集有关：latin1=1、gbk=2、utf8=3、utf8mb4=4

    ​	char(n)：n*字符集长度

    ​	varchar(n)：n * 字符集长度 + 2字节

  - ​	数值类型

    ​	TINYINT：1个字节

    ​	SMALLINT：2个字节

    ​	MEDIUMINT：3个字节

    ​	INT、FLOAT：4个字节

    ​	BIGINT、DOUBLE：8个字节

  - 时间类型

    DATE：3个字节

    TIMESTAMP：4个字节

    DATETIME：8个字节

  - 字段属性

    NULL属性占用1个字节，如果一个字段设置了NOT NULL，则没有此项。Extra

- **Extra**

  ​	Extra表示很多额外的信息，各种操作会在Extra提示相关信息，常见几种如下：
  - Using where

    ​	表示查询需要通过索引回表查询数据。

  - Using index

    ​	表示查询需要通过索引，索引就可以满足所需数据。

  - Using fifilesort

    ​	表示查询出来的结果需要额外排序，数据量小在内存，大的话在磁盘，因此有Using fifilesort建议优化。

  - Using temprorary

    ​	查询使用到了临时表，一般出现于去重、分组等操作。

### **3.2** **回表查询**

在之前介绍过，InnoDB索引有聚簇索引和辅助索引。聚簇索引的叶子节点存储行记录，InnoDB必须要有，且只有一个。辅助索引的叶子节点存储的是主键值和索引字段值，通过辅助索引无法直接定位行记录，通常情况下，需要扫码两遍索引树。先通过辅助索引定位主键值，然后再通过聚簇索引定位行记录，这就叫做**回表查询**，它的性能比扫一遍索引树低。

总结：通过索引查询主键值，然后再去聚簇索引查询记录信息

### **3.3** **覆盖索引**

在MySQL官网，类似的说法出现在explain查询计划优化章节，即explain的输出结果Extra字段为Usingindex时，能够触发索引覆盖。

![image-20211016181331431](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.10.16/image-20211016181331431-4379213.png)

**只需要在一棵索引树上就能获取SQL所需的所有列数据，无需回表，速度更快，这就叫做索引覆盖**。

实现索引覆盖最常见的方法就是：将被查询的字段，建立到组合索引。

### **3.4** **最左前缀原则**

复合索引使用时遵循最左前缀原则，最左前缀顾名思义，就是最左优先，即查询中使用到最左边的列，那么查询就会使用到索引，如果从索引的第二列开始查找，索引将失效。

![image-20211016181637835](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.10.16/image-20211016181637835-4379399.png)

### **3.5 LIKE** **查询**

**面试题：MySQL在使用like模糊查询时，索引能不能起作用？**

回答：MySQL在使用Like模糊查询时，索引是可以被使用的，只有把%字符写在后面才会使用到索引。

​	select * from user where name like '%o%'; //不起作用

​	select * from user where name like 'o%'; //起作用

​	select * from user where name like '%o'; //不起作用

### **3.6 NULL** **查询**

**面试题：如果MySQL表的某一列含有NULL值，那么包含该列的索引是否有效？** 

对MySQL来说，NULL是一个特殊的值，从概念上讲，NULL意味着“一个未知值”，它的处理方式与其他值有些不同。比如：不能使用=，<，>这样的运算符，对NULL做算术运算的结果都是NULL，count时不会包括NULL行等，NULL比空字符串需要更多的存储空间等。

“NULL columns require additional space in the row to record whether their values are NULL. For MyISAM tables, each NULL column takes one bit extra, rounded up to the nearest byte.”

NULL列需要增加额外空间来记录其值是否为NULL。对于MyISAM表，每一个空列额外占用一位，四舍五入到最接近的字节。

![image-20211016182025342](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.10.16/image-20211016182025342-4379627.png)

虽然MySQL可以在含有NULL的列上使用索引，但NULL和其他数据还是有区别的，不建议列上允许为NULL。最好设置NOT NULL，并给一个默认值，比如0和 ‘’ 空字符串等，如果是datetime类型，也可以设置系统当前时间或某个固定的特殊值，例如'1970-01-01 00:00:00'。

### **3.7** **索引与排序**

MySQL查询支持fifilesort和index两种方式的排序，fifilesort是先把结果查出，然后在缓存或磁盘进行排序操作，效率较低。使用index是指利用索引自动实现排序，不需另做排序操作，效率会比较高。

fifilesort有两种排序算法：双路排序和单路排序。

双路排序：需要两次磁盘扫描读取，最终得到用户数据。第一次将排序字段读取出来，然后排序；第二次去读取其他字段数据。

单路排序：从磁盘查询所需的所有列数据，然后在内存排序将结果返回。如果查询数据超出缓存sort_buffffer，会导致多次磁盘读取操作，并创建临时表，最后产生了多次IO，反而会增加负担。解决方案：少使用select *；增加sort_buffffer_size容量和max_length_for_sort_data容量。

如果我们Explain分析SQL，结果中Extra属性显示Using fifilesort，表示使用了fifilesort排序方式，需要优化。如果Extra属性显示Using index时，表示覆盖索引，也表示所有操作在索引上完成，也可以使用index排序方式，建议大家尽可能采用覆盖索引。

- 以下几种情况，会使用index方式的排序。

  - ORDER BY 子句索引列组合满足索引最左前列

    ```sql
    explain select id from user order by id; //对应(id)、(id,name)索引有效
    ```

  - WHERE子句+ORDER BY子句索引列组合满足索引最左前列		

    ```sql
    explain select id from user where age=18 order by name; //对应 (age,name)索引
    ```

- 以下几种情况，会使用fifilesort方式的排序。
  - 对索引列同时使用了ASC和DESC

    ```sql
    explain select id from user order by age asc,name desc; //对应 (age,name)索引
    ```

  - WHERE子句和ORDER BY子句满足最左前缀，但where子句使用了范围查询（例如>、<、in等）

    ```sql
    explain select id from user where age>10 order by name; //对应 (age,name)索引
    ```

  - ORDER BY或者WHERE+ORDER BY索引列没有满足索引最左前列

    ```sql
    explain select id from user order by name; //对应(age,name)索引
    ```

  - 使用了不同的索引，MySQL每次只采用一个索引，ORDER BY涉及了两个索引

    ```sql
    explain select id from user order by name,age; //对应(name)、(age)两个索引
    ```

  - WHERE子句与ORDER BY子句，使用了不同的索引

    ```sql
    explain select id from user where name='tom' order by age; //对应 (name)、(age)索引
    ```

  - WHERE子句或者ORDER BY子句中索引列使用了表达式，包括函数表达式

    ```sql
    explain select id from user order by abs(age); //对应(age)索引
    ```

## 4. **查询优化**

### **4.1** **慢查询定位**

- **开启慢查询日志**

  ​	查看 MySQL 数据库是否开启了慢查询日志和慢查询日志文件的存储位置的命令如下：

  ```sql
  SHOW VARIABLES LIKE 'slow_query_log%'
  ```

  ​	通过如下命令开启慢查询日志：

  ```sql
  SET global slow_query_log = ON; 
  SET global slow_query_log_file = 'OAK-slow.log'; 
  SET global log_queries_not_using_indexes = ON; 
  SET long_query_time = 10;
  ```

  long_query_time：指定慢查询的阀值，单位秒。如果SQL执行时间超过阀值，就属于慢查询记录到日志文件中。

  log_queries_not_using_indexes：表示会记录没有使用索引的查询SQL。前提是slow_query_log的值为ON，否则不会奏效。

- **查看慢查询日志**

  - ​	文本方式查看

    ​	直接使用文本编辑器打开slow.log日志即可。

    ​	time：日志记录的时间

    ​	User@Host：执行的用户及主机

    ​	Query_time：执行的时间

    ​	Lock_time：锁表时间

    ​	Rows_sent：发送给请求方的记录数，结果数量

    ​	Rows_examined：语句扫描的记录条数

    ​	SET timestamp：语句执行的时间点

    ​	select....：执行的具体的SQL语句

  - 使用mysqldumpslow查看

    MySQL 提供了一个慢查询日志分析工具mysqldumpslow，可以通过该工具分析慢查询日志内容。

    在 MySQL bin目录下执行下面命令可以查看该使用格式。

    ```shell
    perl mysqldumpslow.pl --help
    ```

    运行如下命令查看慢查询日志信息：	

    ```shell
    perl mysqldumpslow.pl -t 5 -s at C:\ProgramData\MySQL\Data\OAK-slow.log
    ```

    除了使用mysqldumpslow工具，也可以使用第三方分析工具，比如pt-query-digest、mysqlsla等。

### **4.2** **慢查询优化**

**索引和慢查询**

- 如何判断是否为慢查询？

  MySQL判断一条语句是否为慢查询语句，主要依据SQL语句的执行时间，它把当前语句的执行时间跟 long_query_time 参数做比较，如果语句的执行时间 > long_query_time，就会把这条执行语句记录到慢查询日志里面。long_query_time 参数的默认值是 10s，该参数值可以根据自己的业务需要进行调整。

- 如何判断是否应用了索引？

  SQL语句是否使用了索引，可根据SQL语句执行过程中有没有用到表的索引，可通过 explain命令分析查看，检查结果中的 key 值，是否为NULL。

- 应用了索引是否一定快？

  下面我们来看看下面语句的 explain 的结果，你觉得这条语句有用上索引吗？比如

  ```
  select * from user where id>0;
  ```

  虽然使用了索引，但是还是从主键索引的最左边的叶节点开始向右扫描整个索引树，进行了全表扫描，此时索引就失去了意义。

  而像 select * from user where id = 2; 这样的语句，才是我们平时说的使用了索引。它表示的意思是，我们使用了索引的快速搜索功能，并且有效地减少了扫描行数。

查询是否使用索引，只是表示一个SQL语句的执行过程；而是否为慢查询，是由它执行的时间决定的，也就是说是否使用了索引和是否是慢查询两者之间没有必然的联系。

我们在使用索引时，不要只关注是否起作用，应该关心索引是否减少了查询扫描的数据行数，如果扫描行数减少了，效率才会得到提升。对于一个大表，不止要创建索引，还要考虑索引过滤性，过滤性好，执行速度才会快。

**提高索引过滤性**

假如有一个5000万记录的用户表，通过sex='男'索引过滤后，还需要定位3000万，SQL执行速度也不会很快。其实这个问题涉及到索引的过滤性，比如1万条记录利用索引过滤后定位10条、100条、1000条，那他们过滤性是不同的。索引过滤性与索引字段、表的数据量、表设计结构都有关系。

下面我们看一个案例：

```sql
表：student 
字段：id,name,sex,age 
造数据：insert into student (name,sex,age) select name,sex,age from student; 
SQL案例：select * from student where age=18 and name like '张%';（全表扫描）
```

优化1 

```sql
alter table student add index(name); //追加name索引
```

优化2

```sql
alter table student add index(age,name); //追加age,name索引
```

优化3

```sql
//可以看到，index condition pushdown 优化的效果还是很不错的。再进一步优化，我们可以把名 字的第一个字和年龄做一个联合索引，这里可以使用 MySQL 5.7 引入的虚拟列来实现。 
//为user表添加first_name虚拟列，以及联合索引(first_name,age) 
alter table student add first_name varchar(2) generated always as (left(name, 1)), add index(first_name, age); explain select * from student where first_name='张' and age=18;
```

慢查询原因总结

​	全表扫描：explain分析type属性all

​	全索引扫描：explain分析type属性index

​	索引过滤性不好：靠索引字段选型、数据量和状态、表设计

​	频繁的回表查询开销：尽量少用select *，使用覆盖索引

### **4.3** **分页查询优化**

- 一般性分页

  一般的分页查询使用简单的 limit 子句就可以实现。limit格式如下：

  ```sql
  SELECT * FROM 表名 LIMIT [offset,] rows
  ```

  - 第一个参数指定第一个返回记录行的偏移量，注意从0开始；

  - 第二个参数指定返回记录行的最大数目；

  - 如果只给定一个参数，它表示返回最大的记录行数目；

  **思考1：如果偏移量固定，返回记录量对执行时间有什么影响？**

  ```sql
  select * from user limit 10000,1; 
  select * from user limit 10000,10; 
  select * from user limit 10000,100; 
  select * from user limit 10000,1000;
  select * from user limit 10000,10000;	
  ```

  结果：在查询记录时，返回记录量低于100条，查询时间基本没有变化，差距不大。随着查询记录量越大，所花费的时间也会越来越多。

  **思考2：如果查询偏移量变化，返回记录数固定对执行时间有什么影响？**	

  ```sql
  select * from user limit 1,100; 
  select * from user limit 10,100; 
  select * from user limit 100,100; 
  select * from user limit 1000,100; 
  select * from user limit 10000,100; 
  ```

  结果：在查询记录时，如果查询记录量相同，偏移量超过100后就开始随着偏移量增大，查询时间急剧的增加。（这种分页查询机制，每次都会从数据库第一条记录开始扫描，越往后查询越慢，而且查询的数据越多，也会拖慢总查询速度。）

- 分页优化方案

  **第一步：利用覆盖索引优化**

  ```sql
  select * from user limit 10000,100; 
  select id from user limit 10000,100;
  ```

  **第二步：利用子查询优化**

  ​	

  ```sql
  select * from user limit 10000,100; 
  select * from user where id>= (select id from user limit 10000,1) limit 100;
  ```

  原因：使用了id做主键比较(id>=)，并且子查询使用了覆盖索引进行优化。