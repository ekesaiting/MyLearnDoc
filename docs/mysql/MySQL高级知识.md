# MySQL学习笔记

> 参考链接 

MySQL-InnoDB-MVCC多版本并发控制 https://segmentfault.com/a/1190000012650596

MySQL大表优化方案 https://segmentfault.com/a/1190000006158186

Mysql锁机制简单了解 https://blog.csdn.net/qq_34337272/article/details/80611486

Mysql Innodb 中的锁 https://zhuanlan.zhihu.com/p/31875702

MySQL Explain详解：https://www.cnblogs.com/xuanzhi201111/p/4175635.html

一张图彻底搞懂MySQL的 explain https://juejin.im/post/6844904035900719111

mysql binlog应用场景与原理深度剖析 https://cloud.tencent.com/developer/article/1444390

## 1. 锁

### 1.1 锁是什么？

**锁是计算机协调多个进程或线程并发访问某一资源的机制**。锁保证数据并发访问的**一致性、有效性**；锁冲突也是影响数据库**并发访问**性能的一个重要因素。锁是Mysql在服务器层和存储引擎层的的并发控制。

加锁是消耗资源的，锁的各种操作，包括获得锁、检测锁是否是否已解除、释放锁等。

MySQL 不同的存储引擎支持不同的锁机制，所有的存储引擎都以自己的方式显现了锁机制，服务器层完全不了解存储引擎中的锁实现

### 1.2 锁的分类

#### 1.2.1 按照锁的粒度分类

**行级锁：**只针对当前操作的行（不仅仅是一行数据）进行加锁，
**页面锁：**开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般。



