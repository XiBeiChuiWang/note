# HDFS概述

## HDFS简介

### 产生背景

随着数据量越来越大，一台物理机或者是一个操作系统已经装不下所有数据，因此需要使用分布式文件管理系统，HDFS就是其中的一种

### 定义

一种文件系统，适合一次写入，多次读出的场景

## 优缺点

### 优点

高容错性，数据自动保存多个副本

适合处理大数据

可构建在廉价机器

### 缺点

做不到的ms级的存储数据

无法高效的对大量小文件进行存储

不支持并发写、文件随机修改，仅支持文件追加

