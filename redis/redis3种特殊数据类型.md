# redis3种特殊数据类型

1. ### Geospatial

   > 朋友的定位，附近的人，打车距离计算
   >
   > 底层是 `Zset`，**即可以使用Zset的命令操作Geospatial**

   1. ##### 添加地理位置

      - 两极无法添加
      - 经度：-180 ~ 180（度）
      - 纬度：-85.05112878 ~ 85.05112878（度）

       **`geoadd` Geospatial 经度 纬度 名称** 

   2. ##### 获取指定位置的地理位置

       **`geopos` Geospatial 名称** 

   3. ##### 返回两个给定位置之间的距离（直线距离）

      单位：

      - m ：米
      - km ：千米
      - mi ： 英里
      - ft ：英尺

       **`geodist` Geospatial** 

   4. ##### 以给定值为半径，以经度和维度为中心，查找

      - 附近的人（获得所有附近的人的地址（开启定位））通过半径查询

       **`georadius` Geospatial 经度 纬度 半径 单位** 

   5. ##### 以给定值为半径，以成员(城市名)为中心，查找

       **`georadiusbymember` Geospatial 成员名 半径 单位** 

   6. ##### 返回一个或多个位置元素的geohash表示

      - 如果两个字符串越相似，表示两个地方越近

       **`geohash` Geospatial 成员1 成员2** 

2. ### Hyperloglog

   >  基数统计的算法 
   >
   > 基数：集合中元素的个数（先去重），如{1,2,2,3} 其基数为3（集合去重后为1,2,3 有3个元素）
   >
   > 网页的UV（一个人访问访问一个网站多次，但是还是算作一个人）

   优点

   - 占用内存是**固定的**，2<sup>64</sup>不同的元素的基数，只需要12KB的内存。（大数据情况下，有0.81%错误率）

    **创建一组元素 ： `pfadd` Hyperloglogele1 ele2 ele3 …** 

    **统计对应key的基数：`pfcount` Hyperloglog1 Hyperloglog2 … // 多个key 就是统计这些key并集的基数** 

    **合并：`pfmerge` destkey sourceKey1 sourceKey2 [sourceKey3 …]** 

   [详情可见](https://www.cnblogs.com/linguanh/p/10460421.html)

3. ### Bitmaps

   >  位存储，位图（操作二进制） 

   ##### 案例：一周打卡记录

   1.  **`setbit` Bitmapsoffset bit** 

   2.  **查看单天打卡情况** 

       **`getbit` Bitmapsoffset offset** 

   3.  **统计所有打卡的天数** 

       **`bitcount` Bitmapsoffset** 