## Redis 

> Redis三种模式 https://juejin.im/post/6844904178154897415

> Redis集群的原理和搭建  https://juejin.im/entry/6844903486895685645



## 主从复制

主要是做读写分离与数据备份，但没有高可用，当主节点宕机后需要人工手动选择新的节点作为主节点`SLAVEOF no one`,效率不高，一般搭配哨兵与集群模式使用



Redis 的高可用方案有两种，哨兵和集群。

### 哨兵模式

> https://juejin.im/post/6844903663362637832

主从模式的一种提升，如果只是使用主从模式，当主节点宕机时需要手动选择新的主节点，使用哨兵模式将会为每一个节点（包括主节点与从节点）创建一个新的进程监控该节点并与其他哨兵节点通信。当主节点宕机时，哨兵节点将根据一定规则选举新的节点作为新的主节点，保证服务的可用。哨兵模式的开销较小，有三个节点就可以保证服务的基本可靠

### 集群模式

要让集群正常工作至少需要3个主节点，一般至少要创建6个redis节点，其中三个为主节点，三个为从节点

集群由一系列独立的redis服务其组成，Redis 集群使用数据分片（sharding）而非一致性哈希（consistency hashing）来实现。一个集群包含16384个哈希槽（hash slot）， 数据库中的每个键都属于这 16384 个哈希槽的其中一个， 集群使用公式 CRC16(key) % 16384 来计算键 key 属于哪个槽，如果集群有三台服务器，他们可以平分hash槽 如：

* A服务器：0~5500
* B服务器 5501~11000
* C服务器11001~16384

无论客户端向哪一台服务器发送一个请求，都将转发到对应hash槽的服务器，如client向B服务器发送一个hash slot值为1111的键，他并不会存储或查找该键，而是交给A服务器处理

如果一个redis服务器需要下线，可以手动将它他所管理的slot将交给其他redis服务器管理，如果一个新的redsi服务器加入集群，可以手动将其他redis服务器移出一部分slot给他管理。

#### 集群中的主从复制

集群中每一个redsi节点都拥有一个或多个从节点，从节点平时并不工作，可以读但不能写，只是作为一个主节点的备份，当一个节点和他的所有从节点都宕机时，整个集群都将不可用。

**集群中从节点存在的意义：**

* 作为主服务器的数据备份;
* 在 redis.config 配置允许的情况下，在主服务器故障时候会触发 [自动故障转移] ，升级为主服务器;
* 在 redis.config 配置允许的情况下，主服务器裸奔后，可以自动从其他主服务器的从服务器中迁移一台过来成为裸奔主服务器的从服务器。