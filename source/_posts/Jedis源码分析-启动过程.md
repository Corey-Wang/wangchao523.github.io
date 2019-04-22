---
title: Jedis源码分析-启动过程
date: 2018-12-04 11:19:30
tags: 技术, redis
---
# 基础知识
> Jedis为Redis for java驱动的一种开源实现方式，其中可以支持单机版Redis、Redis Cluster版，Redis Sentinel模式（哨兵）
> 使用了JedisCluster，JedisCluster 为Jedis的cluster版驱动实现，以下为jedis 2.7.0版本的源码
> hashslots：哈希槽，16384个，和Redis Cluster集群版保持一致，每个slot会对应到一个Redis Master
# 初始化做了什么
项目配置了一个IP:PORT，Jedis会通过cluster nodes命令来获取Redis集群的信息
Redis集群的信息包含：节点连接池（Nodes，每个节点都会初始化一个JedisPool），哈希槽对应的节点（Slots，每个哈希槽都会对应到节点连接池）
初始化过程源码分析
## #JedisCluster
DEFAULT_MAX_REDIRECTIONS = 5；// 重要的参数，在获取链接时的重试次数
JedisSlotBasedConnectionHandler初始化时会调用 initializeSlotsCache(nodes, poolConfig);
initializeSlotsCache会调用JedisClusterInfoCache.discoverClusterNodesAndSlots() 来初始化nodes，slots
![](https://ws4.sinaimg.cn/large/006tNbRwly1fxujijkr9kj31b00pcgoj.jpg)

## #JedisClusterInfoCache
nodes map 为  ip:port  ==> JedisPool (这个节点的连接池)， 数量为Redis Cluster节点数量
slots map 为 [0 – 16383] ==> JedisPool，每个哈希槽对应节点的连接池
![](https://ws4.sinaimg.cn/large/006tNbRwly1fxujq5tweej30xq03u3yn.jpg)

## #获取Cluster Nodes，对应redis-cli -c cluster nodes
保存到变量localNodes中
![](https://ws1.sinaimg.cn/large/006tNbRwly1fxujqe1nsbj31nm0u0ah3.jpg)

分析cluster nodes里面包含的slot
![](https://ws2.sinaimg.cn/large/006tNbRwly1fxujqk9f3lj31gs0mqdid.jpg)

Slot只会对应到Master节点，slave节点会被忽略掉
SLOT_INFORMATIONS_START_INDEX = 8;
![](https://ws3.sinaimg.cn/large/006tNbRwly1fxujqokenvj31ws0mm0vv.jpg)


Slave节点的Slot处理
![](https://ws4.sinaimg.cn/large/006tNbRwly1fxujquk24hj31y40cu76g.jpg)

初始化完之后的内容
![](https://ws2.sinaimg.cn/large/006tNbRwly1fxujr21o1bj30s41ba42g.jpg)
