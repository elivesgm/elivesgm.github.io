---
layout: post
title: "Linux Locks"
date: 2017-07-07 18:15:06 
description: "Linux Locks"
tag: Operating System
---

### 1. 死锁
在计算机系统中有很多一次只能由一个进程使用的资源，如打印机，磁带机，一个文件的I节点等。在多道程序设计环境中，若干进程往往要共享这类资源，而且一个进程所需要的资源不止一个。这样，就会出现若干进程竞争有限资源，又推进顺序不当，从而构成无限期循环等待的局面。这种状态就是死锁。

死锁是进程死锁的简称，是由Dijkstra于1965年研究银行家算法时首先提出来的。

所谓死锁，是指多个进程循环等待它方占有的资源而无限期地僵持下去的局面。


#### 1.0 银行家问题

一个银行家如何将一定数目的资金安全地借给若干个客户，使这些客户既能借到钱完成要干的事，同时银行家又能收回全部资金而不至于破产，这就是银行家问题。这个问题同操作系统中资源分配问题十分相似：银行家就像一个操作系统，客户就像运行的进程，银行家的资金就是系统的资源。

一个银行家拥有一定数量的资金，有若干个客户要贷款。每个客户须在一开始就声明他所需贷款的总额。若该客户贷款总额不超过银行家的资金总数，银行家可以接收客户的要求。客户贷款是以每次一个资金单位（如1万RMB等）的方式进行的，客户在借满所需的全部单位款额之前可能会等待，但银行家须保证这种等待是有限的，可完成的。



####1.1 死锁产生条件
从以上分析可见，如果在计算机系统中同时具备下面四个必要条件时，那麽会发生死锁。换句话说，只要下面四个条件有一个不具备，系统就不会出现死锁。

〈1〉互斥条件。即某个资源在一段时间内只能由一个进程占有，不能同时被两个或两个以上的进程占有。这种独占资源如CD-ROM驱动器，打印机等等，必须在占有该资源的进程主动释放它之后，其它进程才能占有该资源。这是由资源本身的属性所决定的。如独木桥就是一种独占资源，两方的人不能同时过桥。

〈2〉不可抢占条件。进程所获得的资源在未使用完毕之前，资源申请者不能强行地从资源占有者手中夺取资源，而只能由该资源的占有者进程自行释放。如过独木桥的人不能强迫对方后退，也不能非法地将对方推下桥，必须是桥上的人自己过桥后空出桥面（即主动释放占有资源），对方的人才能过桥。

〈3〉占有且申请条件。进程至少已经占有一个资源，但又申请新的资源；由于该资源已被另外进程占有，此时该进程阻塞；但是，它在等待新资源之时，仍继续占用已占有的资源。还以过独木桥为例，甲乙两人在桥上相遇。甲走过一段桥面（即占有了一些资源），还需要走其余的桥面（申请新的资源），但那部分桥面被乙占有（乙走过一段桥面）。甲过不去，前进不能，又不后退；乙也处于同样的状况。

〈4〉循环等待条件。存在一个进程等待序列{P1，P2，...，Pn}，其中P1等待P2所占有的某一资源，P2等待P3所占有的某一源，......，而Pn等待P1所占有的的某一资源，形成一个进程循环等待环。就像前面的过独木桥问题，甲等待乙占有的桥面，而乙又等待甲占有的桥面，从而彼此循环等待。

####1.2 死锁预防
银行家算法：

- 系统总共的资源数
- 进程需要的最大资源数
- 进程已经得到的资源数
- 进程还需要的资源数

找到一个系统可以依次满足其需要的所有进程的序列，则该状态就是安全的。参见wikipedia。

####1.3 死锁检测与解除


#### 1.4 pstack和gdb分析死锁
在ubunbu上，使用sudo apt-get install pstack可以直接安装pstack，这里安装上的是二进制的程序。这样安装上的pstack作了测试，无法显示函数的调用关系。根据man pstack的说明，现在的pstack只支持32bit ELF binaries。可以采用[这里的shell脚本](http://nanxiao.me/en/use-pstack-to-track-threads-on-linux/)：
	
	#!/bin/bash
	
	if test $# -ne 1; then
	    echo "Usage: `basename $0 .sh` <process-id>" 1>&2
	    exit 1
	fi
	
	if test ! -r /proc/$1; then
	    echo "Process $1 not found." 1>&2
	    exit 1
	fi
	
	# GDB doesn't allow "thread apply all bt" when the process isn't
	# threaded; need to peek at the process to determine if that or the
	# simpler "bt" should be used.
	
	backtrace="bt"
	if test -d /proc/$1/task ; then
	    # Newer kernel; has a task/ directory.
	    if test `/bin/ls /proc/$1/task | /usr/bin/wc -l` -gt 1 2>/dev/null ; then
	        backtrace="thread apply all bt"
	    fi
	elif test -f /proc/$1/maps ; then
	    # Older kernel; go by it loading libpthread.
	    if /bin/grep -e libpthread /proc/$1/maps > /dev/null 2>&1 ; then
	        backtrace="thread apply all bt"
	    fi
	fi
	
	GDB=${GDB:-/usr/bin/gdb}
	
	if $GDB -nx --quiet --batch --readnever > /dev/null 2>&1; then
	    readnever=--readnever
	else
	    readnever=
	fi
	
	# Run GDB, strip out unwanted noise.
	$GDB --quiet $readnever -nx /proc/$1/exe $1 <<EOF 2>&1 |
	$backtrace
	EOF
	/bin/sed -n \
	    -e 's/^(gdb) //' \
	    -e '/^#/p' \
	    -e '/^Thread/p'

之后就可以成功打印进程的各个线程的调用栈了。一个例子是[2]:

	root@ubuntu:/deadlock# g++ -g lock.cpp -o lock -pthread
	root@ubuntu:/deadlock# ./lock 
	root@ubuntu:/deadlock# ps -ef|grep lock
	root        36     2  0 06:22 ?        00:00:00 [kblockd]
	root      7562  6837  0 17:32 pts/17   00:00:00 ./lock
	root      8018  7445  0 17:42 pts/19   00:00:00 grep --color=auto lock
	root@ubuntu:/deadlock# ./pstack.sh 7562
	Thread 5 (Thread 0x7fb1a16cc700 (LWP 7566)):
	#0  0x00007fb1a2f9c30d in nanosleep () at ../sysdeps/unix/syscall-template.S:84
	#1  0x00007fb1a2f9c25a in __sleep (seconds=0) at ../sysdeps/posix/sleep.c:55
	#2  0x0000000000400a61 in thread4 (arg=0x0) at lock.cpp:80
	#3  0x00007fb1a32a16ba in start_thread (arg=0x7fb1a16cc700)
	#4  0x00007fb1a2fd73dd in clone ()
	Thread 4 (Thread 0x7fb1a1ecd700 (LWP 7565)):
	#0  0x00007fb1a2f9c30d in nanosleep () at ../sysdeps/unix/syscall-template.S:84
	#1  0x00007fb1a2f9c25a in __sleep (seconds=0) at ../sysdeps/posix/sleep.c:55
	#2  0x0000000000400a07 in thread3 (arg=0x0) at lock.cpp:69
	#3  0x00007fb1a32a16ba in start_thread (arg=0x7fb1a1ecd700)
	#4  0x00007fb1a2fd73dd in clone ()
	Thread 3 (Thread 0x7fb1a26ce700 (LWP 7564)):
	#0  __lll_lock_wait () at ../sysdeps/unix/sysv/linux/x86_64/lowlevellock.S:135
	#1  0x00007fb1a32a3dbd in __GI___pthread_mutex_lock (mutex=0x6020a0 <mutex1>)
	#2  0x0000000000400963 in func2 () at lock.cpp:31
	#3  0x00000000004009c6 in thread2 (arg=0x0) at lock.cpp:56
	#4  0x00007fb1a32a16ba in start_thread (arg=0x7fb1a26ce700)
	#5  0x00007fb1a2fd73dd in clone ()
	Thread 2 (Thread 0x7fb1a2ecf700 (LWP 7563)):
	#0  __lll_lock_wait () at ../sysdeps/unix/sysv/linux/x86_64/lowlevellock.S:135
	#1  0x00007fb1a32a3dbd in __GI___pthread_mutex_lock (mutex=0x6020e0 <mutex2>)
	#2  0x0000000000400907 in func1 () at lock.cpp:18
	#3  0x000000000040099f in thread1 (arg=0x0) at lock.cpp:43
	#4  0x00007fb1a32a16ba in start_thread (arg=0x7fb1a2ecf700)
	#5  0x00007fb1a2fd73dd in clone ()
	Thread 1 (Thread 0x7fb1a36bf700 (LWP 7562)):
	#0  0x00007fb1a32a298d in pthread_join (threadid=140400919377664, 
	#1  0x0000000000400b86 in main () at lock.cpp:110
	
	root@ubuntu:/deadlock# gdb - 7562
	Attaching to process 7562
	[New LWP 7563]
	[New LWP 7564]
	[New LWP 7565]
	[New LWP 7566]
	[Thread debugging using libthread_db enabled]
	Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
	0x00007fb1a32a298d in pthread_join (threadid=140400919377664, thread_return=0x0) at pthread_join.c:90
	90	pthread_join.c: No such file or directory.
	(gdb) info thread
	  Id   Target Id         Frame 
	* 1    Thread 0x7fb1a36bf700 (LWP 7562) "lock" 0x00007fb1a32a298d in pthread_join (threadid=140400919377664, thread_return=0x0) at pthread_join.c:90
	  2    Thread 0x7fb1a2ecf700 (LWP 7563) "lock" __lll_lock_wait () at ../sysdeps/unix/sysv/linux/x86_64/lowlevellock.S:135
	  3    Thread 0x7fb1a26ce700 (LWP 7564) "lock" __lll_lock_wait () at ../sysdeps/unix/sysv/linux/x86_64/lowlevellock.S:135
	  4    Thread 0x7fb1a1ecd700 (LWP 7565) "lock" 0x00007fb1a2f9c30d in nanosleep () at ../sysdeps/unix/syscall-template.S:84
	  5    Thread 0x7fb1a16cc700 (LWP 7566) "lock" 0x00007fb1a2f9c30d in nanosleep () at ../sysdeps/unix/syscall-template.S:84
	(gdb) thread 2
	[Switching to thread 2 (Thread 0x7fb1a2ecf700 (LWP 7563))]
	#0  __lll_lock_wait () at ../sysdeps/unix/sysv/linux/x86_64/lowlevellock.S:135
	135	../sysdeps/unix/sysv/linux/x86_64/lowlevellock.S: No such file or directory.
	(gdb) where
	#0  __lll_lock_wait () at ../sysdeps/unix/sysv/linux/x86_64/lowlevellock.S:135
	#1  0x00007fb1a32a3dbd in __GI___pthread_mutex_lock (mutex=0x6020e0 <mutex2>) at ../nptl/pthread_mutex_lock.c:80
	#2  0x0000000000400907 in func1 () at lock.cpp:18
	#3  0x000000000040099f in thread1 (arg=0x0) at lock.cpp:43
	#4  0x00007fb1a32a16ba in start_thread (arg=0x7fb1a2ecf700) at pthread_create.c:333
	#5  0x00007fb1a2fd73dd in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:109
	(gdb) f 2
	#2  0x0000000000400907 in func1 () at lock.cpp:18
	18	    pthread_mutex_lock(&mutex2); 
	(gdb) thread 3
	[Switching to thread 3 (Thread 0x7fb1a26ce700 (LWP 7564))]
	#0  __lll_lock_wait () at ../sysdeps/unix/sysv/linux/x86_64/lowlevellock.S:135
	135	../sysdeps/unix/sysv/linux/x86_64/lowlevellock.S: No such file or directory.
	(gdb) where
	#0  __lll_lock_wait () at ../sysdeps/unix/sysv/linux/x86_64/lowlevellock.S:135
	#1  0x00007fb1a32a3dbd in __GI___pthread_mutex_lock (mutex=0x6020a0 <mutex1>) at ../nptl/pthread_mutex_lock.c:80
	#2  0x0000000000400963 in func2 () at lock.cpp:31
	#3  0x00000000004009c6 in thread2 (arg=0x0) at lock.cpp:56
	#4  0x00007fb1a32a16ba in start_thread (arg=0x7fb1a26ce700) at pthread_create.c:333
	#5  0x00007fb1a2fd73dd in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:109
	(gdb) f 2
	#2  0x0000000000400963 in func2 () at lock.cpp:31
	31	    pthread_mutex_lock(&mutex1); 
	(gdb) p mutex1
	$1 = {__data = {__lock = 2, __count = 0, __owner = 7563, __nusers = 1, __kind = 0, __spins = 0, __elision = 0, __list = {__prev = 0x0, __next = 0x0}}, __size = "\002\000\000\000\000\000\000\000\213\035\000\000\001", '\000' <repeats 26 times>, __align = 2}
	(gdb) p mutex2
	$2 = {__data = {__lock = 2, __count = 0, __owner = 7564, __nusers = 1, __kind = 0, __spins = 0, __elision = 0, __list = {__prev = 0x0, __next = 0x0}}, __size = "\002\000\000\000\000\000\000\000\214\035\000\000\001", '\000' <repeats 26 times>, __align = 2}
	(gdb) 

从两个线程等待的mutex来看，线程7563在等待线程7564占有的mutex2,线程7564又在等待线程7563占有的mutex1，产生了死锁。


### References

[1] [linux 内核的几种锁介绍](http://xilongpei.elastos.org/2014/09/15/linux-lock/),elastos.org.

[2] [一个Linux上分析死锁的简单方法](https://www.ibm.com/developerworks/cn/linux/l-cn-deadlock/),ibm.com.