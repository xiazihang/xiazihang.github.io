---
layout:   post
title:    Redis Cluster 
date:     2016-10-07 23:19:00
author:   "Albert"
tags:
    - Redis  
    - Redis Cluster 
---

> “Redis Cluster是Redis官方在发布3.0版本后内置的一个Redis集群的解决方案，它提供了一个基于多个Redis节点的自动数据分片的解决方案。同样，Redis Cluster还在一定程度上提供了在数据分片期间的可用性保障，在实际的应用场景中，它可以保证我们在丢失一些节点的连接或一些节点故障时仍然能继续操作。不过，在某些比较严重的故障时（比如大部分的master节点不可用时），集群仍然会挂掉。”
- - -  

Redis Cluster
===
在实际应用中，Redis Cluster提供如下功能： 
- 在多节点之间自动实现数据集的分片。 
- 在部分节点出现故障或无法与集群中其他节点通信时仍能保证集群的可用性。  
