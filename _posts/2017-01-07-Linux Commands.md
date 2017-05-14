---
layout: post
title: "Linux Commands"
date: 2017-01-07 18:15:06 
description: "Useful Linux commands"
tag: Linux
---

### 1 查看so符号表

>* objdump -tT libCavium4J.so |grep generateKey

>* nm -D libCavium4J.so  |grep generateKey

>* grep -iao generateKey libCavium4J.so // -i:不区分大小写; -a:; -o
