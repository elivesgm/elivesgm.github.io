---
layout: post
title: "Distributed Systems"
date: 2017-07-30 23:33:06
description: "Tech on Distributed Systems"
tag: Distributed Systems
---

### 1. 集群管理
数据在分布式系统中存在着对分拷贝，以保证HA，但是这也产生了如何保证这些数据一致性的问题。在众多解决方案中，Paxos算法算是最佳算法之一。

无法设计一种分布式协议，使得同事完全具备CAP三个属性。

- Consistency(一致性), CAP理论中的副本一致性特指强一致性；
- Availiability(可用性),指在系统出现异常时可以提供服务；
- Tolerance to the partition of network(分区容忍), 系统可以对网络分区异常进行容错处理。

Notes:

- 强一致性(strong consystency), 任何时刻任何用户或节点都可以读到最近一次成功更新的副本数据。
- Network partition,网络分化：如果某些节点直接通信正常或包率在合理范围内，而节点之间始终无法正常通信
- 分布式系统三态： 成功、失败、超时

#### 1.1 Basics
a. 数据分布方式

**Hash方式进行数据分布**

- 哈希分布数据的可扩展性不高，一旦集群规模需要需要扩展，则几乎所有数据需要被迁移并重新分布
	- 一种解决思路是成倍扩展，只需迁移一半到对应的机器上
	- 另一种是hash值和对应的机器作为元数据由专门的元数据服务器管理
- Hash分布的另一个缺点是，一旦某数据特征出现数据严重不均，容易出现“数据倾斜（data skew)”。
	- 只能重新选择数据特征。

**按数据范围分布**

**按数据量分布**

**一致性Hash**


b. 基本副本协议

**中心化副本控制协议**

由一个中心节点协调副本数据的更新、维护副本之间的一致性。一种常用的中心化副本控制协议是primary-secondary协议。primary-secondary协议中只有一个副本作为primary副本，其他副本作为secondary副本。primary副本作为中心节点负责维护数据的更新、并发控制、协调副本一致性。主要用于解决四类问题：

- 数据更新流程；
- 数据读取方式；
- primary副本的确定和切换，包括确定节点状态(可以用lease机制)和确定新primary(可以用Quorum机制)；
- 数据同步(reconcile)，同步的原因和解决办法：
	- 网络分化，通过回访primary上的操作日志解决;
	- 脏数据(由于primary副本没有进行某一更新操作，而secondary副本反而执行多余的操作而造成secondary副本数据错误)，通过设计不产生(产生概率低)脏数据的分布式协议;
	- secondary是新增的副本而没有数据，通过直接拷贝primary副本解决。

e.g., GFS

**去中心化副本控制协议**

去中心化协议的最大缺点是协议过程比较复杂。Paxos是唯一在工程中得到应用的强一致性去中心化副本控制协议。

e.g., zookeeper使用基于Paxos的去中心化协议选出primary节点，完成primary节点选举之后，转为中心化副本控制协议，即由primary节点复杂同步更新操作到secondary节点。


c. lease机制 

heartbeat + 等待lease过期安全进行primary切换

lease选择：太短(e.g., 1s)可能造成网络瞬断时节点收不到lease而引起服务不稳定; 太长(1min)则在由节点宕机异常时需要等待较长的lease超期。工程中常选择的lease时长是10s级别。

d. Quorum机制

当某次跟新操作Wi一旦在所有N个副本中的W个副本上成功就称该更新操作为成功提交的更新操作，对应的数据为成功提交的数据。令R>N-W，由于Wi仅在W个副本上成功，座椅在读取数据时，最多需要读取R个副本则一定能读到Wi更新后的数据Vi。

Quorum机制确定最新成功已提交数据：

- 要求只有在前一个更新操作成功提交后才可以提交后一个更新操作，版本号必须是连续增加的。
- 读取R个副本，对于这其中版本号最高的数据：
	- 如果已经存在W个，则该数据为最新成功提交数据；
	- 少于W个，则继续读取，读到W个则为最新成功提交数据，否则R中版本号第二大的数据为最新成功提交的副本数据。


#### 1.1 Paxos
集群中各个节点相互以提议的方式通信（对一项数据的修改），提议中带有不断增加的ID号，节点优先同意当前ID最大的提议，并拒绝其他提议，当提议被一半以上节点同意之后，提议便被所有节点接受。

a. Paxos协议中三类节点

- proposer, 提出value
- acceptor, 有N个，proposer提出的value必须获得超过半数(N/2+1)的acceptor批准后才能通过
- learner, 学习被批准的value,至少需要读取超过半数(N/2+1)的acceptor才能学习到一个通过的value

b. 协议流程

Paxos协议按照一轮一轮进行，每轮有一个编号。每轮可能批准一个value，也可能无法批准一个value。每轮分为两阶段, 准备阶段和批准阶段。


#### 1.2 Apache Zookeeper
Paxos的很多具体的解决方案中，最著名的便是Apache Zookeeper。 zookeeper可以用来存储配置数据、实现集群Master选举、分布式锁等。

zxid: epoch + count, 一个leader对应一个唯一的epoch，每次提议进行新的leader选举时epoch都会增加；每个leader任期内产生的更新操作对应一个唯一的有序的count。

zookeeper中每个节点既是proposer也是acceptor。每个节点都有各自最后commit的zxid，代表了这个节点的数据版本。

超过quorum半数的基地那中持有最大zxid的节点会成为新的leader。如果每个节点的zxid都一样则可能无法选举出leader。zookeeper要求为每个节点配置不同的节点编号，记为nodeid,paxos过程以b=(zxid, nodeid)发起提议，从而当zxid相同时，选举nodeid较大的节点为leader.


#### 1.3 Etcd
go语言实现的类似zookeeper的项目。


### 2. RPC
RPC中两个重要的技术点：IO和序列化技术。

#### 2.1 IO

常见的IO模型有阻塞IO、非阻塞IO和异步IO。

#### 2.2 序列化
序列化技术性能的好坏主要包含两方面的含义：一个是序列化时占用的资源（CPU、内存、所需时间）；另一个是序列化之后数据的大小。SOAP WebService 和 REST WebService 通常会把数据序列化成 XML 格式或者 JSON 格式。这两种格式因为都是文本格式，所以有着良好的可读性，但是对于需要频繁使用的远程调用来说，它们的体积偏大。所以边有了性能更好的序列化解决方案，被大家所熟知的有 Protocol Buffers 和 Apache Arvo。


### 3. 消息中间件

#### 3.1 Apache Kafka


#### 3.2 RabbitMQ




### 4. 分布式文件系统
#### 4.1 块存储与对象存储

#### 4.2 Ceph


### 5. 分布式数据库
#### 5.1 关系型数据库

#### 5.2 NoSql

### 6. 负载均衡

#### 6.1 



### References

[1] [刘杰, 分布式系统原理介绍](blog.sciencenet.cn/home.php?mod=attachment&id=31413).

[2] [分布式系统开发之技术介绍](https://my.oschina.net/lifany/blog/423082),oschina.net.

[3] [分布式系统](http://feisky.xyz/distributed/),feisky.xyz.

[4] 