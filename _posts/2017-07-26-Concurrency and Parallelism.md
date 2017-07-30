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






### References

[1] [man](http://man7.org/linux/man-pages/index.html), man7.org.

[2] [使用 C++11 编写 Linux 多线程程序](https://www.ibm.com/developerworks/cn/linux/1412_zhupx_thread/index.html), ibm.com.

