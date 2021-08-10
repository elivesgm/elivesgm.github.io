---
layout: post
title: "LockFreeQueue"
date: 2020-04-29 18:15:06 
description: "Lock"
tag: LOCK
---

## 1.原子操作
- CAS
- Fetch and Add, https://en.wikipedia.org/wiki/Fetch-and-add
- Test and Set, https://en.wikipedia.org/wiki/Test-and-set
- Test and test-and-set, https://en.wikipedia.org/wiki/Test_and_test-and-set

### 1.1 CAS
Compare & Set，或是 Compare & Swap。现在几乎所有的CPU指令都支持CAS的原子操作，X86下对应的是 CMPXCHG 汇编指令。这个操作用C语言来描述就是下面这个样子：（代码来自Wikipedia的Compare And Swap词条）意思就是说，看一看内存*reg里的值是不是oldval，如果是的话，则对其赋值newval。
```c
int compare_and_swap (int* reg, int oldval, int newval)
{
  int old_reg_val = *reg;
  if (old_reg_val == oldval) {
     *reg = newval;
  }
  return old_reg_val;
}
```

CAS的ABA问题
所谓ABA（见维基百科的ABA词条），问题基本是这个样子：
- 进程P1在共享变量中读到值为A
- P1被抢占了，进程P2执行
- P2把共享变量里的值从A改成了B，再改回到A，此时被P1抢占。
- P1回来看到共享变量里的值没有被改变，于是继续执行。
虽然P1以为变量值没有改变，继续执行了，但是这个会引发一些潜在的问题。ABA问题最容易发生在lock free 的算法中的，CAS首当其冲，因为CAS判断的是指针的值。很明显，值是很容易又变成原样的。

### 1.2 内存屏障
内存屏障（英语：Memory barrier），也称内存栅栏，内存栅障，屏障指令等，是一类同步屏障指令，它使得 CPU 或编译器在对内存进行操作的时候, 严格按照一定的顺序来执行, 也就是说在memory barrier 之前的指令和memory barrier之后的指令不会由于系统优化等原因而导致乱序。

大多数现代计算机为了提高性能而采取乱序执行，这使得内存屏障成为必须。

语义上，内存屏障之前的所有写操作都要写入内存；内存屏障之后的读操作都可以获得同步屏障之前的写操作的结果。因此，对于敏感的程序块，写操作之后、读操作之前可以插入内存屏障。

#### 1.2.1 优化屏障 (Optimization Barrier)
现代编译器的代码优化和编译器指令重排可能会影响到代码的执行顺序。编译期指令重排是通过调整代码中的指令顺序，在不改变代码语义的前提下，对变量访问进行优化。从而尽可能的减少对寄存器的读取和存储，并充分复用寄存器。但是编译器对数据的依赖关系判断只能在单执行流内，无法判断其他执行流对竞争数据的依赖关系。就拿无锁环形队列来说，如果Writer做的是先放置数据，再更新索引的行为。如果索引先于数据更新，Reader就有可能会因为判断索引已更新而读到脏数据。即，编译器会对生成的可执行代码做一定优化，造成乱序执行甚至省略（不执行）。

避免编译器的重排序优化操作，保证编译程序时在优化屏障之前的指令不会在优化屏障之后执行。IA-32/AMD64架构上，在Linux下常用的GCC编译器上，优化屏障定义为（linux kernel, include/linux/compiler-gcc.h）：
```c
#define barrier() __asm__ __volatile__("": : :"memory")
```
内存信息已经修改，屏障后的寄存器的值必须从内存中重新获取优化屏障告知编译器：必须按照代码顺序产生汇编代码，不得越过屏障

#### 1.2.2 内存屏障 (Memory Barrier)
1）CPU乱序问题
CPU还有乱序执行（Out-of-Order Execution）的特性。流水线（Pipeline）和乱序执行是现代CPU基本都具有的特性。机器指令在流水线中经历取指、译码、执行、访存、写回等操作。为了CPU的执行效率，流水线都是并行处理的，在不影响语义的情况下。处理器次序（Process Ordering，机器指令在CPU实际执行时的顺序）和程序次序（Program Ordering，程序代码的逻辑执行顺序）是允许不一致的，即满足As-if-Serial特性。显然，这里的不影响语义依旧只能是保证指令间的显式因果关系，无法保证隐式因果关系。即无法保证语义上不相关但是在程序逻辑上相关的操作序列按序执行。从此单核时代CPU的Self-Consistent特性在多核时代已不存在，多核CPU作为一个整体看，不再满足Self-Consistent特性。例如，单核时代的无锁环形队列在多核CPU中，一个CPU核心上的Writer写入数据，更新index后。另一个CPU核心上的Reader依靠这个index来判断数据是否写入的方式不一定可靠。index有可能先于数据被写入，从而导致Reader读到脏数据。

2）Cache一致性问题
前文提到的都是顺序一致性（Sequential Consistency）的问题，没有涉及Cache一致性（Cache Coherence）的问题。
开始提到Cache一致性协议之前，先介绍两个名词：
- Load/Read CPU读操作，是指将内存数据加载到寄存器的过程
- Store/Write CPU写操作，是指将寄存器数据写回主存的过程
现代处理器的缓存一般分为三级，由每一个核心独享的L1、L2 Cache，以及所有的核心共享L3 Cache组成：

由于Cache的容量很小，一般都是充分的利用局部性原理，按行/块来和主存进行批量数据交换，以提升数据的访问效率。既然各个核心之间有独立的Cache存储器，那么这些存储器之间的数据同步就是个比较复杂的事情。缓存数据的一致性由缓存一致性协议保证。这里比较经典的当属MESI协议。Intel的处理器使用从MESI中演化出的MESIF协议，而AMD使用MOESI协议。传统的MESI协议中有两个行为的执行成本比较大。一个是将某个Cache Line标记为Invalid状态，另一个是当某Cache Line当前状态为Invalid时写入新的数据。所以CPU通过Store Buffer和Invalidate Queue组件来降低这类操作的延时。

3）内存屏障
内存屏障 (Memory Barrier)分为写屏障（Store Barrier）、读屏障（Load Barrier）和全屏障（Full Barrier），其作用有两个：
- 防止指令之间的重排序
- 保证数据的可见性
关于第一点，关于指令重排，这里不考虑架构的话，Load和Store两种操作会有Load-Store、Store-Load、Load-Load、Store-Store这四种可能的乱序结果。上文提到的三种屏障则是限制这些不同乱序的机制。
关于第二点。写屏障会阻塞直到把Store Buffer中的数据刷到Cache中；读屏障会阻塞直到Invalid Queue中的消息执行完毕。以此来保证核间各级数据的一致性。
这里要强调，内存屏障解决的只是顺序一致性的问题，不解决Cache一致性的问题（这是Cache一致性协议的责任，也不需要程序员关注）。

## 2.无锁队列
### 2.1 链表实现

### 2.2 数组实现



https://coolshell.cn/articles/8239.html

https://www.codeproject.com/Articles/153898/Yet-another-implementation-of-a-lock-free-circul

https://zh.wikipedia.org/wiki/%E5%86%85%E5%AD%98%E5%B1%8F%E9%9A%9C

https://www.huaweicloud.com/articles/0f109d8c8bfc384562d7c8498392de5d.html
