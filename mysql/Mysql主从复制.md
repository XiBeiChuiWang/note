# Mysql主从复制

1. ## 修改配置文件

   1. #### 主机

      ```bash
      [mysqld]
      #同一局域网唯一
      server-id=1
      #开启二进制日志功能
      log-bin=mysql-bin
      #复制过滤（不需要备份的数据库）
      binlog-ignore-db=mysql
      #为每个session分配内存
      binlog_cache_size=1M
      #主从复制的格式（mixed,statement,row）
      binlog_format=mixed
      ```

   2. #### 从机

      ```bash
      [mysqld]
      server-id=2
      #开启二进制日志文件，以备作为其他从机的主机时用
      log-bin=mysql-slave-bin
      #relay_log中继日志
      relay_log=edu-mysql-relay-bin
      #复制过滤（不需要备份的数据库）
      binlog-ignore-db=mysql
      #为每个session分配内存
      binlog_cache_size=1M
      #主从复制的格式（mixed,statement,row）
      binlog_format=mixed
      slave_skip_errors=1062
      ```

2. ## 授权

   1. 主机新建用户

      ```bash
      CREATE USER 'slave'@'%' IDENTIFIED BY '123456';
      ```

   2. 授予权限

      ```bash
      GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
      ```

   3. 从机连接

      ```bash
      #主机查看
      show master status;
      #从机执行
      change master to master_host='******', master_user='***', master_password='******', master_port=3306, master_log_file='mysql-bin.000001', master_log_pos= 2830, master_connect_retry=30;
      ```

      **master_port**：Master的端口号，指的是容器的端口号

      **master_user**：用于数据同步的用户

      **master_password**：用于同步的用户的密码

      **master_log_file**：指定 Slave 从哪个日志文件开始复制数据，即上文中提到的 File 字段的值

      **master_log_pos**：从哪个 Position 开始读，即上文中提到的 Position 字段的值

      **master_connect_retry**：如果连接失败，重试的时间间隔，单位是秒，默认是60秒

      在Slave 中的mysql终端执行`show slave status \G`用于查看主从同步状态。

   4. 从机开启

      ```bash
      start slave
      ```

   ok，大功告成。

