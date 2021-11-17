# Hadoop

## 大数据简介

### 概念

大数据(big data)，或称巨量资料，指的是所涉及的资料量规模巨大到无法透过目前主流软件工具，在合理时间内达到撷取、管理、处理、并整理成为帮助企业经营决策更积极目的的资讯。

在[维克托·迈尔-舍恩伯格](https://baike.baidu.com/item/维克托·迈尔-舍恩伯格)及肯尼斯·库克耶编写的《[大数据时代](https://baike.baidu.com/item/大数据时代/15434499)》中大数据指不用随机分析法（[抽样调查](https://baike.baidu.com/item/抽样调查)）这样捷径，而采用所有数据进行分析处理。大数据的5V特点（IBM提出）：[Volume](https://baike.baidu.com/item/Volume/17610592)（大量）、[Velocity](https://baike.baidu.com/item/Velocity/1398152)（高速）、[Variety](https://baike.baidu.com/item/Variety/191328)（多样）、[Value](https://baike.baidu.com/item/Value/2285610)（低价值密度）、[Veracity](https://baike.baidu.com/item/Veracity/19362178)（真实性）

一句话概括：

**大数据解决的是海量数据的采集，存储，和分析计算问题**

### 单位

最小的基本单位是bit，按顺序给出所有单位：bit、Byte、KB、MB、GB、TB、PB、EB、ZB、YB、BB、NB、DB。

它们按照进率1024（2的十次方）来计算：

1 Byte =8 bit

1 KB = 1,024 Bytes = 8192 bit

1 MB = 1,024 KB = 1,048,576 Bytes

1 GB = 1,024 MB = 1,048,576 KB

1 TB = 1,024 GB = 1,048,576 MB

1 PB = 1,024 TB = 1,048,576 GB

1 EB = 1,024 PB = 1,048,576 TB

1 ZB = 1,024 EB = 1,048,576 PB

1 YB = 1,024 ZB = 1,048,576 EB

1 BB = 1,024 YB = 1,048,576 ZB

1 NB = 1,024 BB = 1,048,576 YB

1 DB = 1,024 NB = 1,048,576 BB

### 大数据技术生态体系

[![](https://pic.imgdb.cn/item/613dc60144eaada739380130.jpg)](https://pic.imgdb.cn/item/613dc60144eaada739380130.jpg)

## Hadoop简介

Hadoop是一个分布式系统基础架构

主要解决的是海量数据的存储和分析计算问题

## 特性

#### 高可靠

维护多个数据副本，即使某个节点发生故障，也不会造成数据的丢失

#### 高扩展

可以方便的扩展节点

#### 高效性

在MapReduce的思想下，Hadoop是并行工作的（多个节点配合）

#### 高容错

能自动将失败的任务重新分配

## 组成

#### Mapduce

计算

Map负责分

Reduce负责汇总各个容器计算出的结果

#### Yarn

资源调度

ResourceManager ：集群资源

NodeManager ：单个服务器资源

ApplicationMaster：单个任务

Container：容器，相当于一台独立的服务器

#### HDFS

分布式文件存储

NameNode

DataNode

SecondaryNameNode

#### Common

辅助工具

[![](https://pic.imgdb.cn/item/613dbed144eaada7392d5ea9.jpg)](https://pic.imgdb.cn/item/613dbed144eaada7392d5ea9.jpg)





