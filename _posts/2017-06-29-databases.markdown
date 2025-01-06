---
title: 七周七数据库
date: 2017-06-29
tags: mysql, riak, redis, hbase
---
七周七数据库
--------
[Postgres](#postgres)
[Riak](#riak)
[Rbase](#hbase)
[MongoDB](#mongodb)

### 概述
  * 选取那种数据库能够最好的解决你的问题
  * 数据库类型：关系型(Postgres)、键值类型(Riak, Redis)、多列型(Hbase)、面向文档型(MongoDb, CouchDb)、图型(Neo4j)
  * 模式是数据库所强制的一个刚性框架
  * 实现横向扩展（MongoDB， Hbase、Riak）， 纵向扩展（Postgres、Neo4j、Redis）
  * 关系型数据库： Mysql, H2, HSQLDB,SQLite,
  * 键值数据库， KV， 因为对资源的要求非常少，所以会有着高性能， 但是当你有复杂的查询和聚合的需求时候，不会有帮助
  * 列型数据库：大约介于关系数据库-键值数据库之间HBase
  * 文档型数据库：存储的就是文档， 有一个独一无二的IB标识
  * 图数据库：在数据中建立联系， 如社交网络

## postgres
  1. 庞大的特性集合（触发器、存储过程、高级索引、数据安全性）， 查询灵活，模式是规范的。
  2. 事务保证一致性， 索引，连接查询。
  3. 存储过程， 可以减少客户端代码的编写，提高性能，但是， 却决定与数据库的绑定，
  4. 视图，create View
  5. 全文索引 TSVector TSQuery， Lucence, Sphinx， 搜索引擎

  优点:
   > 符合ACID，确保提交的院子型，一致性， 隔离和持久, 数据模式非常规范，通过优化（建立索引等）可以轻松处理TB的数据

  缺点:
  > 分区不是强项，数据要求比较严格，如果数据过于灵活，不是很容易融入到关系数据库严格的数据模式中，或者不需要一个完整的数据库功能带来的开销，需要进厂非常大量的键值对读写的操作

### riak
  1. 分布式的键值数据库，值可以是任何类型的数据， 普通文本，json，xml，图片，音频等。对互联网友好，可以通过URL，http方式查询
  2. 容错： 服务器可以在任何时刻启动或者停止，而不会引起任何单点故障，（非常注重可以写入性。。。为的是可以回家睡觉）
  3. 缺点：对于自定义的查询缺乏支持， 键值的存储设计，是数据值无法进行连接（外键）

##### Riak 中的架构设计
  1. 环状：个个节点加入到一个环， 每个Vnode代表一系列经过hash的键，这个可以通过计算hash可快速寻找键值
  2. Riak中写入对象时候，可以选择在多个节点上创建这个值的副本，如果某个节点发生故障，还有另一个节点的副本可以使用.三个数值， N， W， R， ：n第一次写入最终复制的节点数量， 集群中的副本数量，W， 第一次成功写入返回前，必须成功写入的节点数，R是成功读取一项数据所必须的节点数，
  3. 向量时钟， 解决冲突：
  4. 可以使用Erlang， Javascript，写函数，提交前后的回调函数
  5. 搜索： 如果你打算为大规模web应用提供搜索功能， Riak是一个明确的选择。 但是你需要大量简答的自有定义的查询，则不是明智的

  优点:
  > 致力于最高可用性， 避免单点故障，如果使用Erlang就能够扩展Riak的核心，map reduce

  缺点:
  > 没有严格的数据结构，数据模式， 自定义查询很弱。


### hbase

  优点：
  > 值得注意的特性：健壮的可伸缩架构， 内置的版本，压缩功能。 保存wiki页面，自动实现了版本管理

  缺点：
  > 5个节点是最小的配置， 学习曲线陡峭，除了行键之外，不提供任何索引功能，不提供排序， 所以如果不能使用行键查找，就只能使用扫描表，或者自己维护索引， 数据类型的缺失： 所有字段的数值都作为不解释的字节数组， 比如整数值、字符串、日期之类的，对这些的解释取决玉应用程序。

### mongodb
  1. 文档数据库，允许数据以嵌套形式的状态保存， 没有模式，
  2. 关系数据库有强大的查询能力， Riak， Hbase分布式存储的特点，Mongo在这俩昂着之前找到了最佳结合点， 保存大规模数据， 又能满足自有定义的查询。 是一个json文档数据库，能够无限的嵌套。
  3. 安装一般需要先创建数据存储位置， /data/db, mongod
  4. Mongo不支持join操作
  5. db.users.find(function(){ return this.age > 50 && this.age < 100})使用函数作为find参数的，最后在考虑，因为查询很慢，不能使用索引。
  6. Map Reduce, db.runCommand({mapReduce: 'tables', map: map_fun, reduce: reduce_fun, out: 'result_tables'})
  7. 副本集合， 分片，地理空间， GridFS，

  优点：
  > 能够通过复制和横向伸缩， 处理大量的数据，+ 非常灵活的数据模型，强大的SQL

  缺点：
  >  鼓励反规范化的模式， 将任意类型的数据插入集合中，是很危险的。需要努力去设计和管理。


### redis
    1. redis-server, redis-cli
    2. redis 数据类型，Lists, Sets, Sorted sets, Hashes, Bit arrays, HyperLogLogs
    3. redis keys 
       * 空值可以是key
       * 太长太浪费空间，　太短容易造成冲突
       * 坚持使用一种模式
    4. 命令集合: 
       * set, get, incr, decr, mset, exists, type, expire, ttl
       * list rpush, lpush, lrange, rpop, del, llen, lpop, 
       * hash, hmset, hget, hgetall, 
       * sets, sadd, smembers, sismember, sinter, sunionstore, spop, scard, 
