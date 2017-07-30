---
layout: post
title: "Concurrency and Parallelism"
date: 2016-05-01 18:15:06 
description: "Concurrency and Parrallelism"
tag: Operating System
---

### 0 Concurrency and Parallelism
当一个CPU执行一个线程时，另一个CPU可以执行另一个线程，两个线程互不抢占CPU资源，可以同时进行，这种方式我们称之为并行(Parallel)。区别：并发和并行是即相似又有区别的两个概念，并行是指两个或者多个事件在同一时刻发生；而并发是指两个或多个事件在同一时间间隔内发生。


### 1. 网络编程
#### 1.1 TCP/IP网络编程
Client: 

- socket 
- connect 
- write 
- recv

Server: 

- socket
- bind
- listen
- accept
- read
- send

An example: [Simple TCP/IP C/S](https://github.com/elivesgm/socket/tree/master/tcpip_socket).

a. socket

	#include <sys/socket.h>
	sockfd = socket(int socket_family, int socket_type, int protocol);

b. connect

	#include <sys/types.h>
	#include <sys/socket.h>
	int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);

c. bind

	#include <sys/types.h>
	#include <sys/socket.h>
	int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);

d. listen

	#include <sys/types.h>
	#include <sys/socket.h>
	int listen(int sockfd, int backlog);

Notes： The `backlog` argument defines the maximum length to which the queue of pending connections for sockfd may grow.  If a connection request arrives when the queue is full, the client may receive an error with an indication of ECONNREFUSED or, if the underlying protocol supports retransmission, the request may be ignored so that a later reattempt at connection succeeds.

e. accept

	#include <sys/types.h>
	#include <sys/socket.h>
	int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);



### 2. 多线程编程

- pthread\_create
- pthread\_exit
- pthread\_join
- pthread\_detach

#### 2.1 pthread_create
create a new thread.

	#include <pthread.h>
	int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
	                   void *(*start_routine) (void *), void *arg);

Notes:

- The new thread starts execution by invoking start_routine(); arg is passed as the sole argument of start\_routine().
- Compile and link with -pthread.
- On success, pthread_create() returns 0; on error, it returns an error number, and the contents of *thread are undefined.
- 创建一个线程之后，我们还需要考虑一个问题：该如何处理这个线程的结束？一种方式是等待这个线程结束，在一个合适的地方调用 thread 实例的 join() 方法，调用者线程将会一直等待着目标线程的结束，当目标线程结束之后调用者线程继续运行；另一个方式是将这个线程分离，由其自己结束，通过调用 thread 实例的 detach() 方法将目标线程置于分离模式。一个线程的 join() 方法与 detach() 方法只能调用一次，不能在调用了 join() 之后又调用 detach()，也不能在调用 detach() 之后又调用 join()，在调用了 join() 或者 detach() 之后，该线程的 id 即被置为默认值（空线程），表示不能继续再对该线程作修改变化[2]。


#### 2.2  pthread_exit
terminate calling thread

	#include <pthread.h>
	void pthread_exit(void *retval);

Notes:

- Compile and link with -pthread.
- terminates the calling thread and returns a value via retval that (if the thread is joinable) is available to another thread in the same process that calls pthread_join.

#### 2.3 pthread_join
join with a terminated thread

	#include <pthread.h>
	int pthread_join(pthread_t thread, void **retval);

Notes:

- waits for the thread specified by thread to terminate.  If that thread has already terminated, then pthread_join() returns immediately.  The thread specified by thread must be joinable.
- If retval is not NULL, then pthread_join() copies the exit status of the target thread (i.e., the value that the target thread supplied to pthread_exit(3)) into the location pointed to by retval. If the target thread was canceled, then PTHREAD_CANCELED is placed in the location pointed to by retval.


#### 2.4 pthread_detach
detach a thread

	#include <pthread.h>
	int pthread_detach(pthread_t thread);

Notes:

- marks the thread identified by thread as detached.  When a detached thread terminates, its resources are automatically released back to the system without the need for another thread to join with the terminated thread.





### 3. 异步编程


### 4. IO复用
与多进程和多线程技术相比，I/O多路复用技术的最大优势是系统开销小，系统不必创建进程/线程，也不必维护这些进程/线程，从而大大减小了系统的开销。

#### 4.1 同步IO
a. select

	#include <sys/select.h>
	#include <sys/time.h>
	
	int select(int maxfdp1,fd_set *readset,fd_set *writeset,fd_set *exceptset,const struct timeval *timeout)
	返回值：就绪描述符的数目，超时返回0，出错返回-1

select的几大缺点：

（1）每次调用select，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大
（2）同时每次调用select都需要在内核遍历传递进来的所有fd，这个开销在fd很多时也很大
（3）select支持的文件描述符数量太小了，默认是1024

b. poll

	# include <poll.h>
	int poll ( struct pollfd * fds, unsigned int nfds, int timeout);

poll的机制与select类似，与select在本质上没有多大差别，管理多个描述符也是进行轮询，根据描述符的状态进行处理，但是poll没有最大文件描述符数量的限制。
描述fd集合的方式不同，poll使用pollfd结构而不是select的fd_set结构，其他的都差不多。poll和select同样存在一个缺点就是，包含大量文件描述符的数组被整体复制于用户态和内核的地址空间之间，而不论这些文件描述符是否就绪，它的开销随着文件描述符数量的增加而线性增大。

c. epoll

	#include <sys/epoll.h>
	int epoll_create(int size);
	int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
	int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);

epoll是在2.6内核中提出的，是之前的select和poll的增强版本。相对于select和poll来说，epoll更加灵活，没有描述符限制。epoll使用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件存放到内核的一个事件表中，这样在用户空间和内核空间的copy只需一次。

d. 比较

（1）select，poll实现需要自己不断轮询所有fd集合，直到设备就绪，期间可能要睡眠和唤醒多次交替。而epoll其实也需要调用epoll_wait不断轮询就绪链表，期间也可能多次睡眠和唤醒交替，但是它是设备就绪时，调用回调函数，把就绪fd放入就绪链表中，并唤醒在epoll_wait中进入睡眠的进程。虽然都要睡眠和交替，但是select和poll在“醒着”的时候要遍历整个fd集合，而epoll在“醒着”的时候只要判断一下就绪链表是否为空就行了，这节省了大量的CPU时间。这就是回调机制带来的性能提升。

（2）select，poll每次调用都要把fd集合从用户态往内核态拷贝一次，并且要把current往设备等待队列中挂一次，而epoll只要一次拷贝，而且把current往等待队列上挂也只挂一次（在epoll_wait的开始，注意这里的等待队列并不是设备等待队列，只是一个epoll内部定义的等待队列）。这也能节省不少的开销。

refers to [3].



#### 4.2 异步IO



### References

[1] [man](http://man7.org/linux/man-pages/index.html), man7.org.

[2] [使用 C++11 编写 Linux 多线程程序](https://www.ibm.com/developerworks/cn/linux/1412_zhupx_thread/index.html), ibm.com.

[3] [select、poll、epoll之间的区别总结](http://www.cnblogs.com/Anker/p/3265058.html),cnblogs.com.

