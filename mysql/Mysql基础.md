# Mysql基础

## 事务的ACID

### A（原子性）

##### 概念：

一个事务必须是一个不可分割的最小工作单元，整个事务要么全完成，要么全不完成。

##### 实现原理：

**回滚日志（undo log）** 

实际上，原子性底层就是通过undo log实现的。undo log主要记录了数据的逻辑变化，比如一条INSERT语句，对应一条DELETE的undo log，对于每个UPDATE语句，对应一条相反的UPDATE的undo log，这样在发生错误时，就能回滚到事务之前的数据状态。 

### C（一致性）

##### 概念：

 事务将数据库从一种状态转变为下一种一致的状态。在事务的前后，数据库的完整性约束没有被破坏。 

### I（隔离性）

##### 概念：

事务之间相互隔离

### D（持久性）

##### 概念：

事务一旦提交，那么就是永久性的，不会因为宕机等故障导致数据丢失（外力影响不保证，比如磁盘损害）。持久性是保证了数据库的高可靠性（High Reliability），而不是高可用性(Hign Availability)。高可用性并不能通过事务来保证。 

##### 实现原理：

 MySQL的innoDB存储引擎，使用Redo log保证了事务的持久性。 

我们在写入数据时，为了性能，并不会直接将数据写入磁盘，而是会写入 **buffer**缓冲区中，那么就意味着我们的mysql如果意外宕机，数据就会丢失，这显然不符合我们持久性的要求，因此我们引入 **Redo log**

 Redo log包括两部分，**重做日志缓冲（redo log buffer**）和**重做日志文件（redo log file）**，前者是易失的缓存，后者是持久化的文件。 

我们写入数据时，会向**redo log buffer**写入，然后在合适的时间刷回磁盘。

看到这，有些同学可能会问了，哎，这不也是会造成数据丢失吗？

不要急我们先看看这张图

![](https://pic.imgdb.cn/item/6070fecb8322e6675cf5f629.jpg)

 mysql支持三种将redo log buffer写入redo log file的时机，可以通innodb_flush_log_at_trx_commit参数配置，各参数值含义如下： 

![](https://pic.imgdb.cn/item/6070ff488322e6675cf6442a.png)

##### **redo log记录形式：**

前面说过，redo log实际上记录数据页的变更，而这种变更记录是没必要全部保存，因此redo log实现上采用了大小固定，循环写入的方式，当写到结尾时，会回到开头循环写日志。

## mvcc

mvcc是基于隐式字段，undo log和read view的

每行记录除了我们自定义的字段外，还有数据库隐式定义的DB_TRX_ID,DB_ROLL_PTR,DB_ROW_ID等字段

- DB_TRX_ID
   6byte，最近修改(修改/插入)事务ID：记录创建这条记录/最后一次修改该记录的事务ID
- DB_ROLL_PTR
   7byte，回滚指针，指向这条记录的上一个版本（存储于rollback segment里）
- DB_ROW_ID
   6byte，隐含的自增ID（隐藏主键），如果数据表没有主键，InnoDB会自动以DB_ROW_ID产生一个聚簇索引
- 实际还有一个删除flag隐藏字段, 既记录被更新或删除并不代表真的删除，而是删除flag变了