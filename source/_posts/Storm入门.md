---
title: Storm入门
date: 2017-03-21 12:45:34
tags: storm,bigdata
---
## 简介
- 免费且开源的实时计算系统
- 简单可靠的处理流式数据，而Hadoop是使用的批处理
- 不受开发语言的限制（thrift）


## hadoop/spark/storm等对比

 * | hadoop M/R | spark | storm 
---|---|---|---
存储 | 磁盘 | 内存 | 内存 
数据集 |  现有数据集 | 现有数据集 | 实时 
任务状态 | 作业管理 | 作业管理 | 常住内存 
常用场景 | 离线的复杂的大数据处理 | 离线的快速大数据处理 | 在线的实时的大数据处理

*Spark Streaming和Storm类似*

[read more](http://www.cnblogs.com/snowbook/p/5773562.html)

## storm核心概念
### 官方抽象图 
![image](http://note.youdao.com/yws/res/2424/WEBRESOURCE9b85a55e7196b3a6dc04f95e76dc9f53)
### 集群结构图
![image](http://www.aboutyun.com/data/attachment/forum/201404/15/225641mt3v1okkkrkkp3rk.jpg)

### Topologies
一个完整的计算主体，相当于是一个M/R任务，区别是M/R任务执行完会退出，而topology永远运行（除非被主动kill）

### Streams、tuples
Storm里面最核心的抽象，Streams使用tuple，tuples可以理解为最小的数据类型元组，可以为 integers, longs, shorts, bytes, strings, doubles, floats, booleans, and byte arrays。也支持自定义可序列化的类型

### Spout
数据源，一般是消息队列，将数据发送到Bolts

### Bolts
具体执行数据分析的节点

### Stream groupings
定义了每个bolt的数据发送策略
- shuffle grouping
- fields grouping
- All grouping(广播，所有bolts都会收到)
- etc.

![image](http://www.aboutyun.com/data/attachment/forum/201404/15/225648a4s3a4a4ll2la0l3.jpg)

[read more1](http://blog.itpub.net/29754888/viewspace-1260026/) [read more2](http://storm.apache.org/releases/1.0.3/Concepts.html)

## 数据的一致性，可靠性
- 每个tuple拥有一个messageId
- Acker机制，每个Topology拥有一个Acker，可以追踪messageId
- Spout、Bolt发出数据后通知Acker自己处理的messageId
- 消息处理失败后，Tuple会被重新发出


## 举个例子
