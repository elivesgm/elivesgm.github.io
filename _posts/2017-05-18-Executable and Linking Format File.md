---
layout: post
title: "Executable and Linking Format Files"
date: 2016-05-18 20:15:06 
description: "Executable and Linking Format (ELF) Files"
tag: Linux
---

### 0. 概述

ELF(Executable and Linking Format)是一种对象文件的格式，用于定义不同类型的对象文件(Object files)中都放了什么东西、以及都以什么样的格式去放这些东西。

### 1. 对象文件分类
对象文件(Object files)有三个种类：

1) 可重定位的对象文件(Relocatable file)

这是由汇编器汇编生成的 .o 文件。后面的链接器(link editor)拿一个或一些 Relocatable object files 作为输入，经链接处理后，生成一个可执行的对象文件 (Executable file) 或者一个可被共享的对象文件(Shared object file)。我们可以使用 ar 工具将众多的 .o Relocatable object files 归档(archive)成 .a 静态库文件。

通过file test.o查看sum.o的文件信息，可以看到其实ELF格式文件, eg., test.o: ELF 32-bit LSB relocatable, Intel 80386, version 1 (SYSV), not stripped 通过命令 readelf -h ./test.o可以以ELF格式查看文件test.o。

2) 可执行的对象文件(Executable file)

文本编辑器vi、调式用的工具gdb、播放mp3歌曲的软件mplayer等等都是Executable object file。Linux系统里面，存在两种可执行的东西。除了这里说的 Executable object file，另外一种就是可执行的脚本(如shell脚本)。

3) 可被共享的对象文件(Shared object file)

动态库.so 文件。静态库生成可执行程序，每个生成的可执行程序中都会有一份库代码的拷贝。如果在磁盘中存储这些可执行程序，那就会占用额外的磁盘空间。如果将静态库换成动态库，那么这些问题都不会出现。动态库使用的两个步骤：

链接编辑器(link editor)拿它和其他Relocatable object file以及其他shared object file作为输入，经链接处理后，生成另外的 shared object file或者 executable file。

在运行时，动态链接器(dynamic linker)拿它和一个Executable file以及另外一些 Shared object file 来一起处理，在Linux系统里面创建一个进程映像。