---
layout: post
title: "Perf"
date: 2017-06-15 18:15:06 
description: "Usages about perf"
tag: Tools
---

### 1. perf
#### 1.1 perf
Perf是一个包含22种子工具的工具集，以下是最常用的5种：

- perf list
- perf stat
- perf top
- perf record
- perf report
- perf script

#### 1.2 Flame Graph
整个图形看起来就像一团跳动的火焰，这也正是其名字的由来。燃烧在火苗尖部的就是 CPU 正在执行的操作，不过需要说明的是颜色是随机的，本身并没有特殊的含义，纵向表示调用栈的深度，横向表示消耗的时间。因为调用栈在横向会按照字母排序，并且同样的调用栈会做合并，所以一个格子的宽度越大越说明其可能是瓶颈。综上所述，主要就是看那些比较宽大的火苗，特别留意那些类似平顶山的火苗。

#### 1.4 Example
perf and flame graph:

    root@HSH1000041305:/usr/bin# ps -eo pid,cmd |grep sshd |sort -k4 |awk '{print $1}'
    2237
    61597
    91943
    root@ubuntu:/usr/bin# ps ax|fgrep sshd
      2237 ?        Ss    0:00 /usr/sbin/sshd -D
    61597 ?        S      0:00 sshd: ubuntu@pts/0 
    91943 ?        Ss    0:00 sshd: ftpuser [priv]
    root@ubuntu:/usr/bin# perf record -e cpu-clock -p 61597 -o sshd.data -F 2000 -g sleep 3
    [ perf record: Woken up 1 times to write data ]
    [ perf record: Captured and wrote 0.015 MB sshd.data (~652 samples) ]
    root@ubuntu:/usr/bin# perf script -i sshd.data >sshd.details
    root@ubuntu:/usr/bin# perf script -i sshd.data >sshd.raw
    root@ubuntu:/usr/bin# stackcollapse-perf.pl sshd.raw | flamegraph.pl >sshd.svg
    bash: /usr/bin/stackcollapse-perf.pl: Permission denied
    bash: /usr/bin/flamegraph.pl: Permission denied
    root@ubuntu:/usr/bin# chmod +x stackcollapse-perf.pl flamegraph.pl
    root@ubuntu:/usr/bin# ll stackcollapse-perf.pl flamegraph.pl
    -rwxr-xr-x 1 root root 25703 Jun 15 17:28 flamegraph.pl*
    -rwxr-xr-x 1 root root  6463 Jun 15 17:28 stackcollapse-perf.pl*
    root@ubuntu:/usr/bin# stackcollapse-perf.pl sshd.raw | flamegraph.pl >sshd.svg
  
使用浏览器打开sshd.svg查看。


### References
[1] [Perf](https://perf.wiki.kernel.org/index.php/Main_Page), kernel.org.

[2] [flamegraphs](http://www.brendangregg.com/flamegraphs.html), brendangregg.com.

[3] [白话火焰图](https://huoding.com/2016/08/18/531), huoding.com.
