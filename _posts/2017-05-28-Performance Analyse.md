---
layout: post
title: "Performance Analyse"
date: 2017-05-28 09:35:13 
description: "Introduction of Performance Analyse Tools"
tag: Tools
---

### 0. 性能热点分析工具
- [perf](https://perf.wiki.kernel.org/index.php/Main_Page), 由linux内核携带并且同步更新，基本能满足日常使用。
- [oprofile](http://oprofile.sourceforge.net/news/), 一个较过时的性能检测工具了，基本被perf取代，命令使用起来也不太方便。比如opcontrol --no-vmlinux , opcontrol --init等命令启动，然后是opcontrol --start， opcontrol --dump， opcontrol -h停止，opreport查看结果等，一大串命令和参数。有时候使用还容易忘记初始化，数据就是空的。 
- [gprof](https://sourceware.org/binutils/docs/gprof/), 主要是针对应用层程序的性能分析工具，缺点是需要重新编译程序，而且对程序性能有一些影响。不支持内核层面的一些统计，优点就是应用层的函数性能统计比较精细，接近我们对日常性能的理解，比如各个函数时间的运行时间，，函数的调用次数等，很人性易读。
- [systemtap](https://sourceware.org/systemtap/), 是一个运行时程序或者系统信息采集框架，主要用于动态追踪，当然也能用做性能分析，功能最强大，同时使用也相对复杂。不是一个简单的工具，可以说是一门动态追踪语言。**如果程序出现非常麻烦的性能问题时，推荐使用systemtap**。

### 1. Perf

Perf是一个包含22种子工具的工具集，以下是最常用的5种[1]：

- perf-list
- perf-stat
- perf-top
- perf-record
- perf-report


perf有一个缺点就是不直观。火焰图就是为了解决这个问题。它能够以矢量图形化的方式显示事件热点及函数调用关系。 


### References

[1] [系统级性能分析工具—Perf](http://blog.csdn.net/zhangskd/article/details/37902159), csdn.

[2] [Perf -- Linux下的系统性能调优工具](https://www.ibm.com/developerworks/cn/linux/l-cn-perf1/), ibm.

[3] [初识火焰图](http://yangxikun.github.io/linux%E6%80%A7%E8%83%BD%E5%88%86%E6%9E%90/2016/09/24/flame-graph.html), github.io.
