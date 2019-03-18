---
layout: post
title: "Operating System"
date: 2017-05-15 23:15:06 
description: "Operating System"
tag: Operating System
---

### 0 国内外现状

- MIT XV6 JOS


### 1 操作系统基础
主要关注x86-32，即80386。

#### 1.1 x86-32硬件 运行模式

有四种运行模式：**实模式**、**保护模式**、**SMM模式**和**虚拟8086模式**。

- **实模式**，80386加电启动后处于实模式运行状态，在这种状态下软件可以访问的物理内存空间不能超过1MB，且无法发挥Intel 80386以上级别的32为CPU的4GB内存管理能力。早期DOS就运行在实模式。
- **保护模式**，支持内存分页机制，提供了对虚拟内存的良好支持。保护模式下80386支持多任务，还支持有限级机制，不同的程序可以运行在不同的优先级上。优先级一共分为0~34个级别，操作系统运行在最高的优先级0上，应用程序则运行在比较低的级别上；配合良好的检查机制后，既可以在任务间实现数据的安全共享也可以很好地隔离各个任务。

#### 1.2 x86-32硬件 内存架构

- 地址是访问内存空间的索引。
- 80386是32位的处理器，可以寻址的物理内存地址空间为2^32=4G字节。
- 物理内存地址空间是处理器提交到总线上用于访问计算机系统中的内存和外设的最终地址。一个计算机系统中只有一个物理地址空间。
- 线性地址空间是在操作系统的虚存管理之下内个运行的应用程序能访问的地址空间。每个运行的应用程序都认为自己独享整个计算机系统的地址空间，这样可让多个运行的应用程序之间相互隔离。
- 逻辑地址空间，是应用程序直接使用的地址空间。

    操作系统段机制启动、页机制未启动：逻辑地址->段机制处理->线性地址=物理地址
    操作系统段机制和页机制都启动：逻辑地址->段机制处理->线性地址->页机制处理->物理地址

#### 1.3 x86-32硬件 寄存器

80386寄存器可以分为8组：

- 通用寄存器
- 段寄存器
- 指令指针寄存器
- 标志寄存器
- 控制寄存器
- 系统地址寄存器，调试寄存器，测试寄存器

##### a. 通用寄存器
- EAX: 累加器 add
- EBX: 基址寄存器 base
- ECX: 计数器 count
- EDX: 数据寄存器 data
- ESI: 源地址指针寄存器 source
- EDI: 目的地址指针寄存器 destination
- EBP: 基址指针寄存器 base pointer
- ESP: 堆栈指针寄存器 stack pointer

##### b. 段寄存器

主要用于寻址

- CS: 代码段(code segment)
- DS: 数据段(data segment)
- ES: 附加数据段(extra segment)
- SS: 堆栈段(stack segment)
- FS: 附加段
- GS: 附加段




##### c. 指令寄存器和标志寄存器
- EIP： 指令指针寄存器（也就是PC）， 低16位就是8086的IP，存储的是下一条要执行的指令的内存地址，在分段地址转换中，表示指令的段内偏移地址。
- EFLAGS: 标志寄存器，IF（Interrupt Flag）为中断允许标志位由CLI，STI两条指令来控制；设置IF使CPU可识别外部（可屏蔽）中断请求。复位IF则禁止中断。IF对不可屏蔽外部中断和故障中断的识别没有任何作用。CF,PF,ZF...

### 2 系统启动、中断、异常和系统调用

#### 2.1 系统启动

启动时，系统进入实模式，地址总线只有16位, CS和IP都是16位，PC=CS<<4+IP，可以访问2^20字节=1M字节内存。

BIOS就包含在这1M空间里。在BIOS里有完成系统启动所需支持的功能：基本输入输出程序，系统设置信息、开机自检程序、系统自启动程序等。

BOIS：

- 将加载程序从磁盘的引导扇区(512字节)加载到0x7c00（BIOS只需要识别磁盘上的加载程序）
- 跳转到CS:IP=0000：7C00

加载程序：

- 将操作系统的代码和数据从硬盘加载到内存中（加载程序和操作系统都在磁盘上，加载程序识别磁盘上的文件系统并加载系统）
- 跳转到操作系统的起始地址

总体来讲，系统启动就是系统加电，BIOS读磁盘上的加载程序，加载程序继续读磁盘上的操作系统内核。但是现代操作系统往往有多个磁盘，且每个磁盘有多个扇区。因此系统启动过程就多了两步：
>* ->系统加电BIOS初始化硬件
>* ->**BIOS读取磁盘主引导记录**
>* ->**主引导记录(MBR)代码读取活动分区的引导扇区代码**
>* ->引导扇区代码读取文件系统的加载程序
>* ->加载程序读操作系统内核代码

其中，主引导记录一共有512字节：
>* 启动代码有446字节：检查分区表正确性；加载并跳转到磁盘上的引导程序
>* 硬盘分区表有64字节：描述分区状态和位置；每个分区描述信息占据16字节
>* 分区结束标志有2字节：值为0x55AA，标志一个合法分区

活动分区格式：
>* JMP：跳转指令，跳转到启动代码，与平台相关
>* 文件卷头结构：文件系统描述信息
>* 启动代码：跳转到加载程序
>* 结束标志：0x55AA

加载程序(bootloader)：
>* 从文件系统中读取启动配置信息
>* 可选的操作系统内核列表和加载参数
>* 依据配置加载执行内核并跳转到内核执行

注1：bootloader会使能**保护模式**(将CR0的bit0置为1)和段机制，并从硬盘上读取kernel in ELF格式的OS kernel(跟在MBR后面的扇区）并放到内存中固定位置，跳转到OS的入口点执行，这是控制权交到了OS中。 

注2：ELF(Executable and Linking Format)是一种对象文件的格式，具体参见“[Executable and Linking Format (ELF) Files](/2016/05/Executable-and-Linking-Format-File/)”。

#### 2.2 中断、异常和系统调用

为什么需要中断、异常和系统调用

- 内核是可以信任的第三方
- 只有内核可以执行特权指令
- 方便应用程序（连接外设、内核处理异常、应用程序不影响系统内核）

操作系统和外界的交互靠**中断**、**异常**和**系统调用**。

**中断**：包括硬中断(来自硬件设备的处理请求，eg., 串口、网卡、时钟...)和软中断(通常用于系统调用, eg., INT n)；

**异常**：非法指令或者其他原因导致当前**指令执行失败后**的处理请求，eg., 内存出错、访问地址非法；

**系统调用**：应用程序**主动**向操作系统发出的服务请求，用户程序通过系统调用访问OS内核服务（使用Trap, 也称软中断），eg., 读写磁盘, printf()会触发系统调用write()。

三种常用的应用程序编程接口(对上应用程序提供接口，对下调用系统调用)：Win32 API(windows), POSIX API(Unix/Linux), Java API(JVM)

系统调用&函数调用

**系统调用**

- 使用INT和IRET指令，堆栈切换(应用程序使用用户态堆栈、系统调用使用内核态堆栈)和特权级的转换。

**函数调用**

- 使用CALL和RET常规调用，没有堆栈切换。

其中，函数调用过程包括以下步骤：

- 参数入栈： 将参数从右向左依次压入系统栈
- 函数返回地址入栈：将当前代码区调用指令的下一条指令入栈，供返回继续执行
- 代码区跳转：从当前代码区跳转到被调用函数的入口处
- 栈帧调整

大致指令为：

- push参数3 参数2 参数1
- call函数地址(call指令完成两项工作：首先保存返回地址压入当前指令地址的下一指令地址，在跳转到所调用的函数入口处)

通常函数返回值保存在寄存器EAX中。



### 3. 内存管理

#### 3.1 Basics
Cache line可以简单的理解为CPU Cache中的最小缓存单位。目前主流的CPU Cache的Cache Line大小都是64Bytes。假设我们有一个512字节的一级缓存，那么按照64B的缓存单位大小来算，这个一级缓存所能存放的缓存个数就是512/64 = 8个[4]。

#### 3.2 连续内存分配

- 内碎片：分配单元内部未使用的内存
- 外碎片：分配单元之间未使用的内存

#### 3.3 非连续内存分配
##### 3.3.1 段式存储管理
程序可移植，移植程序的时候只需要更改段基址。

把进程的地址空间分为若干个段：

- 栈，堆栈段，存放临时变量，函数参数，函数返回值
- 堆，malloc的内存
- 代码段，代码
- 数据段，存放已初始化的全局变量和静态变量
- BSS段，存放未初始化的全局变量和静态变量，使用前会被系统自动清0。

##### 3.3.2 页式存储管理
- 便于内存管理和减少碎片
- 虚拟内存管理

把地址空间分为页帧



##### 3.3.3 页表

a. 快表, Translation Look-aside Buffer, TLB

利用CPU中的缓存，把近期访问过的表项缓存到CPU中。找到则不需要访问内存，直接访问物理地址。找不到则去内存查找，查到之后缓存到CPU。


b. 多级页表

简介引用的方式减小页表长度

##### 3.3.4 段页式存储管理


#### mmap
mmap是一种内存映射文件的方法，即将一个文件或者其它对象映射到进程的地址空间，实现文件磁盘地址和进程虚拟地址空间中一段虚拟地址的一一对映关系。实现这样的映射关系后，进程就可以采用指针的方式读写操作这一段内存，而系统会自动回写脏页面到对应的文件磁盘上，即完成了对文件的操作而不必再调用read,write等系统调用函数。相反，内核空间对这段区域的修改也直接反映用户空间，从而可以实现不同进程间的文件共享。

refers to [2].


### 4. 进程与线程

### 4.1 进程与线程简介

- http://www.ruanyifeng.com/blog/2013/04/processes_and_threads.html
- http://www.qnx.com/developers/docs/6.4.1/neutrino/getting_started/s1_procs.html

### 4.2 线程池
refers to [3].



### 5. 进程间通信
- 管道，Pipe(Unix IPC)
- 有名管道，Named Pip, or FIFO(Unix IPC)
- 信号，Signal(Unix, Posix, BSD IPC)
- 消息队列，Message queue(System V IPC, Posix IPC)
- 信号量，Semaphores(System V IPC, Posix IPC)
- 共享内存，Shared memory(System V IPC, Posix IPC)
- 套接字，Socket(BSD IPC)

#### 5.1 Posix IPC
- 消息队列
- 信号量
- 共享内存


#### 5.2 管道和FIFO

#### 5.2.1 管道
create pipe

	#include <unistd.h>
	int pipe(int pipefd[2]);

a **unidirectional** data channel that can be used for interprocess communication.  The array pipefd is used to return two file descriptors referring to the ends of the pipe.  pipefd[0] refers to the read end of the pipe.  pipefd[1] refers to the write end of the pipe.  Data written to the write end of the pipe is buffered by the kernel until it is read from the read end of the pipe.



#### 5.2.2 FIFO


### 6. 线程间通信





### References

[1] 操作系统(Operating Systems) (2017), http://os.cs.tsinghua.edu.cn/oscourse/OS2017spring.

[2] [mmap分析](http://www.cnblogs.com/huxiao-tee/p/4660352.html), cnblogs.com.

[3] [简单Linux C线程池](http://www.cnblogs.com/venow/archive/2012/11/22/2779667.html),cnblogs.com.

[4] [All about cpu cache](http://cenalulu.github.io/linux/all-about-cpu-cache/),github.io.
