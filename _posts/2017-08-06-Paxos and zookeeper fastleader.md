---
layout: post
title: "Paxos and zookeeper fastleader"
date: 2017-08-06 23:33:06
description: "Tech on Distributed Systems"
tag: Distributed Systems
---

### 0. Atomic Broadcast
In fault-tolerant distributed computing, an atomic broadcast or total order broadcast is a broadcast where all correct processes in a system of multiple processes deliver the same set of messages in the same order; that is, the same sequence of messages. 

**The broadcast is termed "atomic" because it either eventually completes correctly at all participants, or all participants abort without side effects**.

The following properties are usually required from an atomic broadcast protocol:

- Validity: if a correct participant broadcasts a message, then all correct participants will eventually deliver it.
- Uniform Agreement: if one correct participant delivers a message, then all correct participants will eventually deliver that message.
- Uniform Integrity: a message is delivered by each participant at most once, and only if it was previously broadcast.
- Uniform Total Order: the messages are totally ordered in the mathematical sense; that is, if any correct participant delivers message 1 first and message 2 second, then every other correct participant must deliver message 1 before message 2.


### 1. Paxos
BY Leslie Lamport.

The PAXOS algorithm for solving consensus is used to implement a fault-tolerant Atomic Broadcast.

####1.1 算法内容

Phase 1

(a) A proposer selects a proposal number n and sends a prepare request with number n to a majority of acceptors.

(b) If an acceptor receives a prepare request with number n greater than that of any prepare request to which it has already responded, then it responds to the request with a promise not to accept any more proposals numbered less than n and with the highest-numbered pro-posal (if any) that it has accepted.

Phase 2

(a) If the proposer receives a response to its prepare requests (numbered n) from a majority of acceptors, then it sends an accept request to each of those acceptors for a proposal numbered n with a value v , where v is the value of the highest-numbered proposal among the responses, or is any value if the responses reported no proposals.

(b) If an acceptor receives an accept request for a proposal numbered n, it accepts the proposal unless it has already responded to a prepare request having a number greater than n.

#### 1.2 算法实现libpaxos
节点之间socket通信libevent的组件buffer_event, 将底层的通信接口抽象为缓存操纵，并在其上供给读、写、事务回调


### 2. zookeeper
zookeeper atomic broadcast protocol.


### References
[1] [Paxos Made Simple](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/12/paxos-simple-Copy.pdf),microsoft.com. 

[2] [Paxos made code: Implementing a high throughput Atomic Broadcast
](http://www.inf.usi.ch/faculty/pedone/MScThesis/marco.pdf),usi.ch.

[3] [图解分布式一致性协议Paxos](http://codemacro.com/2014/10/15/explain-poxos/),cppblog.com.

[4] [Paxos算法介绍](https://taozj.org/201611/learn-note-of-distributed-system-(2)-paxos-algorithm.html),taozj.org.

[5] [Paxos算法](https://zh.wikipedia.org/zh-cn/Paxos%E7%AE%97%E6%B3%95#.E5.AE.9E.E4.BE.8B),wikipedia.org.

[6] [Paxos原理、历程及实战](http://chuansong.me/n/2189245),chuansong.me.