# redis事务，配置文件，持久化

### 事务

**在学完数据类型后，我们来看看事务**

**首先我们明确，redis单条命令是保证原子型的，而事务不保证原子性**

**且redis事务没有隔离级别的概念**

##### 事务流程

- 开启事务（`multi`）
- 命令入队（`其他命令`）
- 执行事务（`exec`）
- 取消事务（discard）

##### 异常执行

1. **编译时**

    整个命令队列都不会执行 

2. **运行时**

    报错语句，会抛出异常；其他语句照样运行 

##### redis实现乐观锁

  **使用watch监控**

- 在提交事务时，如果key的value没有发生变化，则成功执行
- 在提交事务时，如果key的value发生了变化，则无法成功执行

##### 用java操控redis

```java
<!--导入jedis包 Redis客户端-->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.3.0</version>
</dependency>
<!--导入fastjson-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.70</version>
</dependency>
    
// 操作事务
    // 连通
Jedis jedis = new Jedis("IP地址", 6379);
jedis.auth("密码");
JSONObject jsonObject = new JSONObject();
jsonObject.put("hello","world");
jsonObject.put("name","liuyou");
jsonObject.put("pwd","密码");
String s = jsonObject.toJSONString();
jedis.flushAll();
/// 加监听 watch
// jedis.watch("user");
// 开启事务
Transaction multi = jedis.multi();
try {
    multi.set("user",s);
    // 其他语句
    // 执行事务
    multi.exec();
} catch (Exception e) {
    // 取消事务
    multi.discard();
    e.printStackTrace();
} finally {
    System.out.println(jedis.get("user"));
    // 关闭连接
    jedis.close();
}
```

### 配置

```bash
#redis.conf 配置项说明如下：

#Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程
daemonize yes
#当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定
pidfile /var/run/redis.pid
#指定Redis监听端口，默认端口为6379，作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字
port 6379
#绑定的主机地址
bind 127.0.0.1
#当 客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能
timeout 300
#指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose
loglevel verbose
#日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null
logfile stdout
#设置数据库的数量，默认数据库为0，可以使用SELECT 命令在连接上指定数据库id
databases 16
#指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合
save seconds changes
#Redis默认配置文件中提供了三个条件：
save 900 1
save 300 10
save 60 10000
#分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。
#指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大
rdbcompression yes
#指定本地数据库文件名，默认值为dump.rdb
dbfilename dump.rdb
#指定本地数据库存放目录
dir ./
#设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步
slaveof <masterip> <masterport>
#当master服务设置了密码保护时，slav服务连接master的密码
masterauth <master-password>
#设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH <password>命令提供密码，默认关闭
requirepass foobared
#设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max number of clients reached错误信息
maxclients 128
#指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区
maxmemory <bytes>
#指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no
appendonly no
#指定更新日志文件名，默认为appendonly.aof
appendfilename appendonly.aof
#指定更新日志条件，共有3个可选值：
no：表示等操作系统进行数据缓存同步到磁盘（快）
always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全）
everysec：表示每秒同步一次（折衷，默认值）
appendfsync everysec
#指定是否启用虚拟内存机制，默认值为no，简单的介绍一下，VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析Redis的VM机制）
vm-enabled no
#虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享
vm-swap-file /tmp/redis.swap
#将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0
vm-max-memory 0
#Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用默认值
vm-page-size 32
#设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘上每8个pages将消耗1byte的内存。
vm-pages 134217728
#设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4
vm-max-threads 4
#设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启
glueoutputbuf yes
#指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法
hash-max-zipmap-entries 64
hash-max-zipmap-value 512
#指定是否激活重置哈希，默认为开启（后面在介绍Redis的哈希算法时具体介绍）
activerehashing yes
#指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件
include /path/to/local.conf
```

### 持久化

1. #### RDB

   - `RDB介绍`
     在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存里；
     RDB保存的是dump.rdb文件；
     Redis会单独创建（`fork`）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失；
     `fork`：复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等）数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程；
   - `如何触发RDB快照`

   1. 配置文件中默认的快照配置（冷拷贝后重新使用：可以`cp dump.rdb dump_new.rdb`）；
   2. 使用命令save或者bgsave
      `save`：save时只管保存，其它不管，全部阻塞；
      `bgsave`：Redis会在后台异步进行快照操作，快照同时还可以响应客户端请求。可以通过`lastsave`命令获取最后一次成功执行快照的时间；
   3. 执行`flushall`命令，也会产生dump.rdb文件，但里面是空的，无意义；

   - `如何恢复`
     将备份文件 (`dump.rdb`) 移动到 redis 安装目录并启动服务即可，通过`config get dir`可获取目录；
   - `如何停止`
     动态所有停止RDB保存规则的方法：`redis-cli config set save ""`；
   - `优势`

   1. 适合大规模的数据恢复；
   2. 对数据完整性和一致性要求不高；

   - `劣势`

   1. 在一定间隔时间做一次备份，所以如果redis意外宕掉的话，就会丢失最后一次快照后的所有修改；
   2. fork的时候，内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑；

2. #### AOF（Append Only File）

   - `AOF介绍`
     以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来(读操作不记录)，只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作（AOF保存的是appendonly.aof文件）；
   - `AOF启动/修复/恢复`

   1. 正常恢复
      启动：修改默认的`appendonly no`，改为`yes`；
      将有数据的aof文件复制一份保存到对应目录(目录通过`config get dir`命令获取)；
      恢复：重启redis然后重新加载；
   2. 异常恢复
      启动：修改默认的`appendonly no`，改为`yes`；
      备份被破坏的aof文件；
      修复：使用`redis-check-aof --fix`命令进行修复；
      恢复：重启redis然后重新加载；

   - `rewrite`

   1. rewrite介绍
      AOF采用文件追加方式，文件会越来越大为避免出现此种情况，新增了重写机制，当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集.可以使用命令`bgrewriteaof`；
   2. 重写原理
      AOF文件持续增长而过大时，会fork出一条新进程来将文件重写(也是先写临时文件最后再rename)，遍历新进程的内存中数据，每条记录有一条的Set语句。重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件，这点和快照有点类似；
   3. 触发机制
      Redis会记录上次重写时的AOF大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发；

   - `优势`
     每修改同步：`appendfsync always` 同步持久化，每次发生数据变更会被立即记录到磁盘 性能较差但数据完整性比较好；
     每秒同步：`appendfsync everysec` 异步操作，每秒记录，如果一秒内宕机，有数据丢失；
     不同步：`appendfsync no` 从不同步；
   - `劣势`

   1. 相同数据集的数据而言aof文件要远大于rdb文件，恢复速度慢于rdb；
   2. aof运行效率要慢于rdb,每秒同步策略效率较好，不同步效率和rdb相同；

   