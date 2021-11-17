# redis订阅发布，主从复制，常见问题

1. ### 订阅发布

   ![](https://pic.imgdb.cn/item/601692f63ffa7d37b3571f31.png)

   ![](https://pic.imgdb.cn/item/601692f63ffa7d37b3571efe.png)

   | 序号 | 命令及描述                                                   |
   | :--- | :----------------------------------------------------------- |
   | 1    | [PSUBSCRIBE pattern [pattern ...\]](https://www.runoob.com/redis/pub-sub-psubscribe.html) 订阅一个或多个符合给定模式的频道。 |
   | 2    | [PUBSUB subcommand [argument [argument ...\]]](https://www.runoob.com/redis/pub-sub-pubsub.html) 查看订阅与发布系统状态。 |
   | 3    | [PUBLISH channel message](https://www.runoob.com/redis/pub-sub-publish.html) 将信息发送到指定的频道。 |
   | 4    | [PUNSUBSCRIBE [pattern [pattern ...\]]](https://www.runoob.com/redis/pub-sub-punsubscribe.html) 退订所有给定模式的频道。 |
   | 5    | [SUBSCRIBE channel [channel ...\]](https://www.runoob.com/redis/pub-sub-subscribe.html) 订阅给定的一个或多个频道的信息。 |
   | 6    | [UNSUBSCRIBE [channel [channel ...\]]](https://www.runoob.com/redis/pub-sub-unsubscribe.html) 指退订给定的频道。 |

2. ### 主从复制

   - ##### Redis的复制能干什么

   > - 读写分离；
   > - 容灾恢复；

   - ##### **Redis复制如何去应用**

   > 1. 配从(库)不配主(库)；
   > 2. 从库配置：执行命令`slaveof 主库IP 主库端口`：
   >    每次与master断开之后，都需要重新连接，除非你配置进redis.conf文件；
   >    执行命令`info replication`查看主从关系；
   > 3. 修改配置文件细节操作：
   >    拷贝多个redis.conf文件；
   >    开启daemonize yes；
   >    修改pid文件名字；
   >    修改指定端口；
   >    修改log文件名字；
   >    修改dump.rdb名字；
   > 4. 常用三招
   >
   > - 一主多仆
   >   一个master两个slave；
   >   **一些问题？**
   >   (1) 切入点问题？slave1、slave2是从头开始复制还是从切入点开始复制?
   >   答：从头开始复制；
   >   (2) 从机是否可以写？set可否？
   >   答：从机不可以写，也就不能set；
   >   (3) 主机shutdown后情况如何？从机是上位还是原地待命?
   >   答：原地待命；
   >   (4) 主机又回来了后，主机新增记录，从机还能否顺利复制？
   >   答：可以；
   >   (5) 其中一台从机宕掉后情况如何？恢复它能跟上主机吗？
   >   答：不能，需要重新建立主从关系；
   > - 薪火相传
   >   上一个Slave可以是下一个slave的Master，Slave同样可以接收其他slaves的连接和同步请求，那么该slave作为了链条中下一个的master,可以有效减轻master的写压力。
   >   中途变更转向:会清除之前的数据，重新建立拷贝最新的。
   > - 反客为主
   >   主机宕掉后，从机升级为主机：
   >   选择一个从机手动执行`slaveof no one`命令变更为主机，其他从机与该主机建立主从关系。

   - ##### Redis复制的原理

   > master接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，master将传送整个数据文件到slave,以完成一次完全同步
   > 全量复制：而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。slave第一次同步为全量复制。
   > 增量复制：master继续将新的所有收集到的修改命令依次传给slave,完成同步
   > 但是只要是重新连接master,第一次完全同步（全量复制)将被自动执行。

   - ##### **哨兵模式(sentinel)**

   > - 哨兵模式简介
   >   反客为主的自动版，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库；
   > - 启动哨兵模式步骤：
   >
   > 1. 自定义的/myredis目录下新建sentinel.conf文件，名字绝不能错；
   > 2. 配置哨兵,填写内容在sentinel.conf文件中配置：
   >    `sentinel monitor 被监控数据库名字(自己起个名字) 127.0.0.1 6379 1`
   >    上面最后一个数字1，表示主机挂掉后salve投票看让谁接替成为主机，得票数多的成为主机；
   > 3. 启动哨兵
   >    执行命令：`redis-sentinel /myredis/sentinel.conf` （目录依照各自的实际情况配置，可能目录不同）；
   >
   > - **问题**
   >   如果之前的master重启回来，会不会双master冲突？
   >   不会造成双冲突，之前的master会成为slave。（大人，时代变了！）

   

3. ### 常见问题

   1. **缓存穿透：**key对应的数据在数据源并不存在，每次针对此key的请求从缓存获取不到，请求都会到数据源，从而可能压垮数据源。比如用一个不存在的用户id获取用户信息，不论缓存还是数据库都没有，若黑客利用此漏洞进行攻击可能压垮数据库。

      一个一定不存在缓存及查询不到的数据，由于缓存是不命中时被动写的，并且出于容错考虑，如果从存储层查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到存储层去查询，失去了缓存的意义。

      **有很多种方法可以有效地解决缓存穿透问题**，**最常见**的则是采用布隆过滤器，将所有可能存在的数据哈希到一个足够大的bitmap中，一个一定不存在的数据会被 这个bitmap拦截掉，从而避免了对底层存储系统的查询压力。**另外也有一个**更为简单粗暴的方法（我们采用的就是这种），如果一个查询返回的数据为空（不管是数据不存在，还是系统故障），我们仍然把这个空结果进行缓存，但它的过期时间会很短，最长不超过五分钟。

       **粗暴方式伪代码：** 

      ```java
      //伪代码
      public object GetProductListNew() {
          int cacheTime = 30;
          String cacheKey = "product_list";
      
          String cacheValue = CacheHelper.Get(cacheKey);
          if (cacheValue != null) {
              return cacheValue;
          }
      
          cacheValue = CacheHelper.Get(cacheKey);
          if (cacheValue != null) {
              return cacheValue;
          } else {
              //数据库查询不到，为空
              cacheValue = GetProductListFromDB();
              if (cacheValue == null) {
                  //如果发现为空，设置个默认值，也缓存起来
                  cacheValue = string.Empty;
              }
              CacheHelper.Add(cacheKey, cacheValue, cacheTime);
              return cacheValue;
          }
      }
      ```

      

   2. **缓存击穿：**key对应的数据存在，但在redis中过期，此时若有大量并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。

      key可能会在某些时间点被超高并发地访问，是一种非常“热点”的数据。这个时候，需要考虑一个问题：缓存被“击穿”的问题。

      

      **使用互斥锁(mutex key)**

      业界比较常用的做法，是使用mutex。简单地来说，就是在缓存失效的时候（判断拿出来的值为空），不是立即去load db，而是先使用缓存工具的某些带成功操作返回值的操作（比如Redis的SETNX或者Memcache的ADD）去set一个mutex key，当操作返回成功时，再进行load db的操作并回设缓存；否则，就重试整个get缓存的方法。

      SETNX，是「SET if Not eXists」的缩写，也就是只有不存在的时候才设置，可以利用它来实现锁的效果。

      ```java
      public String get(key) {
            String value = redis.get(key);
            if (value == null) { //代表缓存值过期
                //设置3min的超时，防止del操作失败的时候，下次缓存过期一直不能load db
            	if (redis.setnx(key_mutex, 1, 3 * 60) == 1) {  //代表设置成功
                     value = db.get(key);
                     redis.set(key, value, expire_secs);
                     redis.del(key_mutex);
               } else {  //这个时候代表同时候的其他线程已经load db并回设到缓存了，这时候重试获取缓存值即可
                     sleep(50);
                     get(key);  //重试
               }
              } else {
                    return value;      
                }
       }
      ```

      

   3. **缓存雪崩：**当缓存服务器重启或者大量缓存集中在某一个时间段失效，这样在失效的时候，也会给后端系统(比如DB)带来很大压力。

      

       与缓存击穿的区别在于这里针对很多key缓存，前者则是某一个key。 

       缓存失效时的雪崩效应对底层系统的冲击非常可怕！大多数系统设计者考虑用加锁或者队列的方式保证来保证不会有大量的线程对数据库一次性进行读写，从而避免失效时大量的并发请求落到底层存储系统上。还有一个简单方案就时将**缓存失效时间分散开**，比如我们可以在原有的失效时间基础上增加一个随机值，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。 