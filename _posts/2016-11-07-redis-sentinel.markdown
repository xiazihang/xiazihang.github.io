---
layout:   post
title:    Redis Sentinel
date:     2016-10-07 23:19:00
author:   "Albert"
tags:
    - Redis  
    - Sentinel 
---

> “Redis哨兵（Sentinel）为Redis集群提供了高可用的保障，在实际应用中，哨兵的使用能让你的Redis部署在遇到一些特定的故障时免除人为的干涉而自动的解决，同样的，Redis哨兵还提供了一些额外的功能，比如说监控（monitoring）、通知（notification）以及作为客户端使用的配置提供方（configuration provider）。”  

- - -  

# Sentinel 概述：  

下面对哨兵提供这些tasks做一个简单地介绍：  
- `Monitoring`：哨兵会定期地（每秒钟一次）检测你的master和slave是否正常工作。  
- `Notification`：当你的某个被哨兵监控的Redis实例出现异常时，哨兵会通过一些自身的API向指定的系统管理员或是其他的程序发送通知。  
- `Automatic failover`：当你的master服务器出现故障时，哨兵会启动`failover`（自动失效转移）程序，由哨兵集群选举出一个leader sentinel来启动这个程序，并从所有的slaves中重新选出一个新的master服务器，其他的slaves都会自动重新配置来连接这个新的master，当你的应用试图连接已经失效的master服务器时，哨兵集群也会向你的应用返回新的master服务器的地址。  

- - - 
# 哨兵的分布式特性（Distributed nature of Sentinel）：

Redis哨兵是一个分布式系统：  
  
哨兵本身的设计就是要让多个哨兵实例共同运行来解决Redis集群的可用性问题，这样做的好处有如下几点：  
1. 只有当多个哨兵实例一致判断所监控的master服务器出现异常时才会启动失效转移程序，这样可以减少因某些原因造成的错误判断的可能性，提升了哨兵监控的准确性。  
2. 哨兵本身也算是一个集群模式，即便是集群中有些哨兵实例不能正常工作，剩下的哨兵也不会受到影响，这增加了系统的健壮性，在一个实现失效转移的系统中，如果这个系统本身就存在单点故障的隐患的话，这个系统可称不上是一个好的系统。  

# Quick Tutuorial  

下面会通过一个简单的例子，来更形象的介绍一下哨兵是如何对redis集群的可用性做出保障的。  
在本例中，我们会在同在本地模拟一个由三个哨兵实例组成的哨兵集群，并由该哨兵集群来监控我们在本地运行的一个Redis主从集群。  
首先，我们先在本地运行一个一主一从的redis集群：  
1. 请先确认本地的redis能正常运行（端口不一定非要6379，可自行设置）  
2. 为方便起见，复制一份你的redis配置文件`redis.conf`为`redis_slave.conf`，并修改其中的port，使其与master区分，然后再打开`slaveof`注释，后面加上master的host和port，我这里就是`127.0.0.1`和`6379`。  

`redis.conf`：  
{% highlight ruby%}  
....
port 6378
slaveof 127.0.0.1 6379
...
{% endhighlight%}  
3. 启动这个slave实例：`redis-server /usr/local/etc/redis_slave.conf`  
![Redis-server-slave](/img/redis-server-slave.png)  






