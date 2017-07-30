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



### References

[1] [分布式系统开发之技术介绍](https://my.oschina.net/lifany/blog/423082),oschina.net.

[2] 

[3] 