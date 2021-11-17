# redis概述(一)

上一节中讲过了nosql的概念，那我们来看看redis具体的操作

首先，我们需要知道，redis是单线程的，因为他的操作全部都发生在内存中，速度很快，设置成多线程的话反而会浪费时间在线程的上下文切换

redis一共有16个数据库，默认使用第0个。

1. 可以使用`select`进行切换

2.  使用 `dbsize`查看数据库大小 

3. 使用 `key *` 查看所有的key（当前库）

4. 清除当前数据库  `flushdb` 

5. 清空所有数据库  `flushall` 

6.  `exists` key 判断key是否存在

7. 设置key的过期时间`expire` key 时间 // 单位时间为s

   ​								 `ttl` key // 查看剩余存活时间

8. 查看当前key的类型 type key

9.  `append` key  Value 字符串追加（String）

10. 获取字符串长度（String）`strlen` key

11. i++(可用于阅读量的增加) `incr` key  (减少 `decr` key)

12. 步长设置  `incrby` key 步长    `decrby` key 步长 

13.  `getrange` key startIndex endIndex 字符串片段 Range （String）

14.  `setrange` key index replaceString 字符串替换 （String）

15.  `setex` key 时间 value // 设置值，带过期时间 

16.  `setnx` key value // 如果不存在，则设置 

17. `mset` k1 v1 k2 v2 …

    `mget` k1 k2 …

    批量设置、批量获取（原子性操作） （String）

18.  `getset` key value // 先get再set（如果不存在，先返回nil，在设置值；如果存在，就先返回原值，再设置新值） 

19.  `rename` key newName 重命名

20.  `randomKey` 返回一个随机key

21. 手动持久化操作

     `save` 阻塞  

    SAVE 直接调用 rdbSave ，阻塞 Redis 主进程，直到保存完成为止。在主进程阻塞期间，服务器不能处理客户端的任何请求。

     `bgsave` 非阻塞 

    BGSAVE 则 fork 出一个子进程，子进程负责调用 rdbSave ，并在保存完成之后向主进程发送信号，通知保存已完成。 Redis 服务器在BGSAVE 执行期间仍然可以继续处理客户端的请求。

    | 命令   | save               | bgsave                             |
    | ------ | ------------------ | ---------------------------------- |
    | IO类型 | 同步               | 异步                               |
    | 阻塞？ | 是                 | 是（阻塞发生在fock()，通常非常快） |
    | 复杂度 | O(n)               | O(n)                               |
    | 优点   | 不会消耗额外的内存 | 不阻塞客户端命令                   |
    | 缺点   | 阻塞客户端命令     | 需要fock子进程，消耗内存           |

    

### 五大数据类型

 Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。 它支持多种类型的数据结构，如 [字符串（strings）](http://www.redis.cn/topics/data-types-intro.html#strings)， [散列（hashes）](http://www.redis.cn/topics/data-types-intro.html#hashes)， [列表（lists）](http://www.redis.cn/topics/data-types-intro.html#lists)， [集合（sets）](http://www.redis.cn/topics/data-types-intro.html#sets)， [有序集合（sorted sets）](http://www.redis.cn/topics/data-types-intro.html#sorted-sets) 与范围查询， [bitmaps](http://www.redis.cn/topics/data-types-intro.html#bitmaps)， [hyperloglogs](http://www.redis.cn/topics/data-types-intro.html#hyperloglogs) 和 [地理空间（geospatial）](http://www.redis.cn/commands/geoadd.html) 索引半径查询。 Redis 内置了 [复制（replication）](http://www.redis.cn/topics/replication.html)，[LUA脚本（Lua scripting）](http://www.redis.cn/commands/eval.html)， [LRU驱动事件（LRU eviction）](http://www.redis.cn/topics/lru-cache.html)，[事务（transactions）](http://www.redis.cn/topics/transactions.html) 和不同级别的 [磁盘持久化（persistence）](http://www.redis.cn/topics/persistence.html)， 并通过 [Redis哨兵（Sentinel）](http://www.redis.cn/topics/sentinel.html)和自动 [分区（Cluster）](http://www.redis.cn/topics/cluster-tutorial.html)提供高可用性（high availability）。 

1. #### String

   常用命令略

   ##### **使用场景：**

   1. 计数器
   2. 统计多单位的数量
   3. 粉丝数
   4. 对象缓存存储

2. #### List

   1. ##### 从头部/尾部<mark>插入</mark>数据，以及数据<mark>显示</mark>

      **`lpush` list value**

      **`rpush` list  value**

      **`lrange` list  0 -1**

      

   2. ##### .从头部/尾部<mark>移除</mark>数据

      **`lpop` list **

      **`rpop` list **

   3. ##### <mark>获取</mark>指定索引的值

       **`lindex` list index** 

   4. ##### 获取列表长度

       **`llen` list ** 

   5. ##### 移除指定的值

       **`lrem` list  count(移除的个数) element** 

   6. ##### 列表修剪 trim

       **`ltrim` list startIndex endIndex** 

   7.  **`lset` key index value // 将列表中 指定index的值替换为对应的value** 

   8. ##### 插入指定的值

       **`linsert` list before|after pivot(那个单词后) value** 

   ##### **使用场景：**

   1. 栈（lpush、lpop）
   2. 队列（lpush、rpop）
      1. 消息队列
      2. 阻塞队列

3. #### Set（集合）

    **集合中的值不能重复（无序）** 

   1. ##### 添加成员到集合中，并查看所有成员

      **`sadd` set member**

      **`smembers` set**

   2. ##### 判定成员是否存在

       **`sismember` set member** 

   3. ##### 查看集合长度（特别）

       **`scard` set**  

   4. ##### 移除指定的成员

       **`srem` set member** 

   5. ##### 获取集合中的随机成员

       **`srandmember` set [count]** 

   6. ##### 随机移除成员

       **`spop` set [count]** 

   7. ##### 移动集合成员到其他集合

       **`smove` source destination member(需要移动的成员)** 

   8. ##### 数字集合类：

      - **差集 `sdiff` set1，set2…**
      - **交集（共同好友）`sinter` set1，set2…**
      - **并集 `sunion` set1 ，set2…**

4. #### Hash

   > key-Map or key-\<k,v>，value是一个Map
   >
   > Hash本质和hash没有区别，只是value变成了Map
   >
   > 用户信息保存，经常变动的信息，适合对象的存储

   1. ##### 简单存储Map和获取Map

      **`hset` hash key value key value**

      **`hget` hash key**

   2. ##### 获取所有Map字段及值

       **`hgetall` hash** 

   3. ##### .删除Map中的字段

       **`hdel` hash  key** 

   4. ##### 查看Map中某字段是否存在

       **`hexists` hash key** 

   5. ##### 获取所有字段或者所有字段对应的值

      **`hkeys` hash** 

      **`hvals` hash** 

   6. ##### 增量i++

       **`hincrby` hash key  value** 

   7. ##### 不存在，就添加成功

       **`hsetnx` hash key value // key在hash 中不存在就添加这个值，否则不做改变** 

5. #### Zset（有序集合）

   > 在Set基础上增加了一个值（用于排序的值）
   >
   > 存储班级成绩表，工资表排序，
   >
   > 普通消息 = 1，重要消息 = 2，带权重进行判断
   >
   > 排行榜应用实现

   1. ##### 添加 和 获取

      **`zadd` zset n value**

      **`zrange` zset startIndex endIndex**

   2. ##### 排序实现（升序和降序）

      **`zrangebyscore` zset  -inf +inf [withscores] // 升序**

      **`zrange` zset 0 -1**

      **`zrevrangebyscore` zset +inf -inf [withscores] // 降序**

      **`zrevrange` zset 0 -1**

      **`zrangebyscore` zset -inf 任意值n // 升序 + 显示区间 [-inf，n]**

   3. ##### 移除指定的值

       **`zrem` zset value** 

   4. ##### 集合的长度

       **`zcard` zset** 

   5. ##### 指定区间的集合长度

       **`zcount` zset startIndex endIndex** 