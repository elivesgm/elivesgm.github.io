---
layout: post
title: "Distributed Systems"
date: 2017-07-30 23:33:06
description: "Tech on Distributed Systems"
tag: Operating System
---

### 1. 集群管理
数据在分布式系统中存在着对分拷贝，以保证HA，但是这也产生了如何保证这些数据一致性的问题。在众多解决方案中，Paxos算法算是最佳算法之一。

#### 1.1 Paxos
集群中各个节点相互以提议的方式通信（对一项数据的修改），提议中带有不断增加的ID号，节点优先同意当前ID最大的提议，并拒绝其他提议，当提议被一半以上节点同意之后，提议便被所有节点接受。

##### 1.1.1 CAP
无法设计一种分布式协议，使得同事完全具备CAP三个属性。

- Consistency(一致性), CAP理论中的副本一致性特指强一致性；
- Availiability(可用性),指在系统出现异常时可以提供服务；
- Tolerance to the partition of network(分区容忍), 系统可以对网络分区异常进行容错处理。


Notes:

- 强一致性(strong consystency), 任何时刻任何用户或节点都可以读到最近一次成功更新的副本数据。
- Network partition,网络分化：如果某些节点直接通信正常或包率在合理范围内，而节点之间始终无法正常通信
- 分布式系统三态： 成功、失败、超时

#### 1.2 Apache Zookeeper
Paxos的很多具体的解决方案中，最著名的便是Apache Zookeeper。 zookeeper可以用来存储配置数据、实现集群Master选举、分布式锁等。

#### 1.2 Etcd
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

#### 6.1 数据分布方式

a. Hash方式进行数据分布

- 哈希分布数据的可扩展性不高，一旦集群规模需要需要扩展，则几乎所有数据需要被迁移并重新分布
	- 一种解决思路是成倍扩展，只需迁移一半到对应的机器上
	- 另一种是hash值和对应的机器作为元数据由专门的元数据服务器管理
- Hash分布的另一个缺点是，一旦某数据特征出现数据严重不均，容易出现“数据倾斜（data skew)”。
	- 只能重新选择数据特征。

b. 按数据范围分布

c. 按数据量分布

d. 一致性Hash




### References

[1] [分布式系统开发之技术介绍](https://my.oschina.net/lifany/blog/423082),oschina.net.

[2] [分布式系统](http://feisky.xyz/distributed/),feisky.xyz.

[3] 