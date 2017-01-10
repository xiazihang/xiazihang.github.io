---
layout:   post
title:    Redis Cluster
date:     2017-01-08 01:29:00
author:   "Albert"
tags:
    - Redis
    - Redis Cluster
---

> “Redis Cluster是Redis官方在发布3.0版本后内置的一个Redis集群的解决方案，它提供了一个基于多个Redis节点的自动数据分片的解决方案。同样，Redis Cluster还在一定程度上提供了在数据分片期间的可用性保障，在实际的应用场景中，它可以保证我们在丢失一些节点的连接或一些节点故障时仍然能继续操作。不过，在某些比较严重的故障时（比如大部分的master节点不可用时），集群仍然会挂掉。”

Redis Cluster
===

在实际应用中，Redis Cluster提供如下功能：

- 在多节点之间自动实现数据集的分片。 
- 在部分节点出现故障或无法与集群中其他节点通信时仍能保证集群的可用性。  

Redis Cluster 通信（TCP ports）
===
每一个Redis集群中的节点都需要两个TCP端口处于开放状态，一个端口用于节点与客户端之间的通信，比如6379。另一个则是用来与集群中其他节点进行通信，这里如果与客户端通信的端口是6379的话，那么它的另一个端口则必须为16379，也就是说两个端口之间的offset必须是10000。在Redis的集群模式中，节点间的通信被称为集群总线（Cluster bus），它们之间采用的是一种特殊的二进制协议。这个集群总线是各节点用来做故障发现，配置更新以及失效转移授权等等操作。客户端是绝不会使用这一个集群总线端口来与节点通信的。所以，一定要在你部署Redis集群的机器上保持这两个端口是畅通的，否则你的Redis集群中的节点将无法进行通信。
集群总线使用的这个特殊的二进制协议，是为了节点与节点间的数据交换，这种协议可以更节省带宽和处理时长。

Redis Cluster 数据分片（data sharding）
===

在数据分片方面，Redis Cluster并没有采用一致性哈希算法，而是采用了另外一种数据分片的格式，叫做哈希槽（`hash slots`）。从概念上来讲，每一个集群中的`key`都会在经过一定的算法后被分配到一个特定的哈希槽中，而这个算法是先将`key`进行[CRC16](https://en.wikipedia.org/wiki/Cyclic_redundancy_check)运算，然后再与`16384`取模。`16384`是Redis官方定义的集群中哈希槽的总数。集群中的所有`master`节点会被平均的分配一定数量的`哈希槽`，比如，你现在有一个Redis集群，包含了三个节点，分别是A、B、C，那么可能出现的`哈希槽`分配情况：

- 节点A：0-5500  
- 节点B：5501-11000  
- 节点A：110001-16383   

这种特性可以让我们很方便地对集群中的节点进行添加或者移除操作。比如，我想在该集群中增加一个D节点，那么我需要将A、B、C三个节点中的`哈希槽`移动一部分到D中来，同样的，如果我想移除A节点，我只需要将A节点所管理的`哈希槽`分配给B和C，当A中的`哈希槽`被全部移走后，节点A就完全地从节点中移除了。这种便利是因为在节点间移动`哈希槽`是不需要重启Redis server的。

Redis Cluster 主从模型（master-slave model）
===

为了使集群在部分master节点挂了或者无法与其他大部分节点通信的情况下仍然保持可用，Redis Cluster采用了master-slave模型，这个主从模型使得master节点中的每一个`哈希槽`会有1到N个备份（N-1个从节点备份）。举个栗子，比如在当前集群中，存在一个master节点A，该master节点挂了三个slave节点，分别是A1，A2和A3，而A节点中包含了一个id是2的哈希槽，那么这个id为2的哈希槽在A1、A2和A3中均会有备份。

Redis Cluster 一致性保证（consistency guarantees）
===

Redis Cluster目前是无法提供数据的强一致性（`strong consistency`）保证的。在实际的应用场景中，这意味着在一些特定情况下，Redis Cluster是会丢失数据的。而造成丢失数据的首要原因是因为Redis Cluster采用的是异步备份（asynchronous replication）的机制。也就是说在写操作时：

- 客户端向主节点发送一条写命令。

- 主节点立即向客户端回复OK。

- 主节点向它的所有从节点发送这一条写操作进行备份。

也就是说主节点在向客户端回复`OK`之前并不会等待它的所有从节点向它回复是否备份成功。那么在主节点向从节点发送写操作备份命令之前就挂了，被选举的新的主节点是没有这条记录的，这样就会永远丢失这一条数据。
为了提高数据的一致性，Redis Cluster提供了同步备份的功能，不过使用这个功能需要考虑到性能的影响，一般来说，在性能和数据一致性之间，总会根据特定的业务场景来做一些权衡取舍。

Redis Cluster 配置参数（configuration parameters）
===

- *cluster-enabled<yes/no>*：是否打开指定Redis实例的集群模式，值为`yes`时，该实例以集群模式启动，否则是单实例模式启动。

- *cluster-config-file<filename>*：集群中节点的配置文件，虽然有指定文件名的选项，不过该文件是不允许手动编辑的，它是在集群启动时，自动为当前节点生成的文件。该文件记录了该节点的状态，持久化变量等等，通常这些文件会在集群状态发生改变时自动更新或者集群下线时从磁盘上删除。

- *cluster-node-time<millseconds>*：集群中节点的最大不可用时间，超过该值即表示该节点不可用。如果是master节点，当超过该值时会被它的从节点failover。

- *appendonly<yes/no>*：Redis中数据持久化的方式，AOF/RDB ??。

Redis Cluster 示例
===
下面通过在本地创建一个Redis集群来更直观地了解Redis Cluster的各种功能。
我们使用配置文件的方式来创建这个Redis的集群，首先列出一个最小可用的配置文件样板：

```code
port 7000 #端口
cluster-enabled yes #以cluster模式启动实例
cluster-config-file nodes.file #指定节点的配置文件
cluster-node-timeout 5000 #指定节点的超时时间
appendonly yes #指定数据持久化的方式
```

如上，便是一个最简单的以cluster模式启动的Redis实例的配置文件。既然我们是要创建一个集群，那么肯定不止需要一个节点，所以，我们首先复制这个文件分别到六个不同的目录下（本例中的Redis集群包含六个Redis节点，三主三从）。目录以端口号命名即可，方便我们切换，别忘了再复制完配置文件后修改他们的端口号，除了端口号以外，其他配置均可保持一致。
在搞定了所有的配置文件之后，让我们在本地的terminal中切出六个窗口来分别启动他们，这样做可以更直观的让我们看到集群中的节点在启动时都会输出什么样的日志。如下图所示：
![redis-clusters](/img/redis-server-01.png)
可以看到，在每个实例启动之后，它们都被分配一个唯一不变的标识符，这个标识符是节点间进行通信的重要的属性。
到目前为止，我们已经成功地将集群中的所有节点都启动起来了，但是，目前它们仍是各自为战，并没有任何的关联，我们还需要利用另外的工具来创建我们的集群，这里推荐Redis官方自带的一个Ruby脚本来创建，文件名是`redis-trib.rb`，它位于Redis源码中的src目录下。

```code
ruby redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
```

当然，要使用这个Ruby脚本的前提是你必须得在你的机器上先安装`redis`这个`gem`包： 

```code
gem install redis
```

如果你连`gem`都没装的话，请自行google。

执行完成之后应该能看到以下输出：

```code
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
127.0.0.1:7000
127.0.0.1:7001
127.0.0.1:7002
Adding replica 127.0.0.1:7003 to 127.0.0.1:7000
Adding replica 127.0.0.1:7004 to 127.0.0.1:7001
Adding replica 127.0.0.1:7005 to 127.0.0.1:7002
M: 3c7784ae915d585e680274696d24e66bb9ae428e 127.0.0.1:7000
   slots:0-5460 (5461 slots) master
M: 73dbda980af150ec2e0442fcee78b17d203befcf 127.0.0.1:7001
   slots:5461-10922 (5462 slots) master
M: 02f080d3f19439b1ef434112a8388a7db16aefd2 127.0.0.1:7002
   slots:10923-16383 (5461 slots) master
S: 1a35e11cc0d18643a1e453a3776f809eb0ce4892 127.0.0.1:7003
   replicates 3c7784ae915d585e680274696d24e66bb9ae428e
S: bc146107083d068d993ed6695ccd59de11386904 127.0.0.1:7004
   replicates 73dbda980af150ec2e0442fcee78b17d203befcf
S: e6ca11f5dcf843a90ce4a3c9dbfb0280ab7bbbf3 127.0.0.1:7005
   replicates 02f080d3f19439b1ef434112a8388a7db16aefd2
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join...
>>> Performing Cluster Check (using node 127.0.0.1:7000)
M: 3c7784ae915d585e680274696d24e66bb9ae428e 127.0.0.1:7000
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: 02f080d3f19439b1ef434112a8388a7db16aefd2 127.0.0.1:7002
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
M: 73dbda980af150ec2e0442fcee78b17d203befcf 127.0.0.1:7001
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: bc146107083d068d993ed6695ccd59de11386904 127.0.0.1:7004
   slots: (0 slots) slave
   replicates 73dbda980af150ec2e0442fcee78b17d203befcf
S: 1a35e11cc0d18643a1e453a3776f809eb0ce4892 127.0.0.1:7003
   slots: (0 slots) slave
   replicates 3c7784ae915d585e680274696d24e66bb9ae428e
S: e6ca11f5dcf843a90ce4a3c9dbfb0280ab7bbbf3 127.0.0.1:7005
   slots: (0 slots) slave
   replicates 02f080d3f19439b1ef434112a8388a7db16aefd2
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

可以很清楚地看到，我们之前启动的六个节点已经被分成了三主三从，并且三个master节点近乎平均地分配了前面提到的那16384个`哈希槽`。现在你再回过头去看看每个节点对应的目录，可以发现多了几个文件： `appendonly.aof`、`nodes.conf` 以及`dump.rdb`。到这一步，我们的Redis集群就创建好了，下面，我们试着来和这个集群进行一些简单的交互。

首先，我们以集群模式开启Redis的客户端，以`7000`端口的这个节点为例：

```code
redis-cli -c -p 7000
```

在此节点上，我们设置一个值： 

```code
127.0.0.1:7000> set foo bar
-> Redirected to slot [12182] located at 127.0.0.1:7002
OK
127.0.0.1:7002>
```

可以看到，Redis Cluster在接受到客户端发出的写命令后，会根据它内部的分片算法，将key（`foo`）进行[CRC16](https://en.wikipedia.org/wiki/Cyclic_redundancy_check)校验，然后再与16384取余，计算出该key所对应的哈希槽（12182），此哈希槽在7002这个端口对应的节点中，所以，Redis Cluster自动的进行重定向，将该key添加到7002的节点中，并同时将客户端的redis实例转到7002的节点下。`get`命令会同样执行以上过程： 

```code
127.0.0.1:7000> get foo
-> Redirected to slot [12182] located at 127.0.0.1:7002
"bar"
127.0.0.1:7002>
```

接下来，我们来试验一下Redis Cluster的可用性情况，我们让7000这个master节点挂掉，可以使用Redis提供的原生命令： 

```code
redis-cli -p 7000 debug segfault
```
我们通过命令： 

```code
redis-cli -p 7002 cluster nodes
```
可以查看当前集群中的各节点的状态： 

```code
bc146107083d068d993ed6695ccd59de11386904 127.0.0.1:7004 slave 73dbda980af150ec2e0442fcee78b17d203befcf 0 1484053015027 5 connected
3c7784ae915d585e680274696d24e66bb9ae428e 127.0.0.1:7000 master,fail - 1484052856076 1484052854416 1 disconnected
02f080d3f19439b1ef434112a8388a7db16aefd2 127.0.0.1:7002 myself,master - 0 0 3 connected 10923-16383
e6ca11f5dcf843a90ce4a3c9dbfb0280ab7bbbf3 127.0.0.1:7005 slave 02f080d3f19439b1ef434112a8388a7db16aefd2 0 1484053015541 6 connected
73dbda980af150ec2e0442fcee78b17d203befcf 127.0.0.1:7001 master - 0 1484053016573 2 connected 5461-10922
1a35e11cc0d18643a1e453a3776f809eb0ce4892 127.0.0.1:7003 master - 0 1484053014508 7 connected 0-5460
```
可以看到，7000端口的状态以及是fail了，它之前的slave节点7003被自动选举为新的master节点，并且维护同样的一组哈希槽，保证了集群在master节点挂掉后，仍能正常工作，并且保证数据不丢失。

待续。 
