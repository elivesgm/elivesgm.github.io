---
layout: post
title: "Memory Leak Detection"
date: 2017-05-27 12:15:01 
description: "Memory Leak Detection Method"
tag: Tools
---

### 0. 内存泄漏工具
- [valgrind](http://valgrind.org/), 一款非常优秀的软件，不需要重新编译程序就能够直接测试。功能也非常强大，能够检测常见的内存错误包括内存初始化、越界访问、内存溢出、free错误等都能够检测出来。valgrind 运行的基本原理是：待测程序运行在valgrind提供的模拟CPU上，valgrind会纪录内存访问及计算值，最后进行比较和错误输出。valgrind也有一个非常大的缺点，就是它会显著降低程序的性能，官方文档说使用memcheck工具时，降低10-50倍。
- [address sanitizer]()，简称asan，是一个用来检测c/c++程序的快速内存检测工具。相比valgrind的优点就是速度快，官方文档介绍对程序性能的降低只有2倍。和valgrind明显不同的是，asan需要添加编译开关重新编译程序，好在不需要自己修改代码。而valgrind不需要编程程序就能直接运行。 address sanitizer集成在了clang编译器中，GCC 4.8版本以上才支持。

