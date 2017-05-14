---
layout: post
title: "Linux Commands"
date: 2017-01-07 18:15:06 
description: "Useful Linux commands"
tag: Linux
---

### 1 查看so符号表

>* objdump -tT libCavium4J.so \|grep generateKey
>* nm -D libCavium4J.so  \|grep generateKey
>* grep -iao generateKey libCavium4J.so // -i:不区分大小写; -a:; -o

### 2 vi/vim常用命令

- /abc, 从光标开始处向文件尾搜索abc, 之后n/N分别表示跳转到下一个/上一个搜索结果；
- ?abc, 从光标开始处向文件首搜索abc；
- n, 在同一方向重复上一次搜索命令 
- N, 在反方向上重复上一次搜索命令 
- :set nu, 显示行号
- :set nonu, 不显示行号
- :234, 跳转到234行

### 3 其他

- Kernel code: http://home.ustc.edu.cn/~boj/courses/linux_kernel/1_boot.html
- Core Dump: http://www.cnblogs.com/hazir/p/linxu_core_dump.html
- Valgrind: http://blog.csdn.net/sduliulun/article/details/7732906