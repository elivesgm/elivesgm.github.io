---
layout: post
title: "select poll and epoll"
date: 2016-04-18 18:15:06 
description: "IO"
tag: Operating system
---

### 0 Basics

https://www.jianshu.com/p/dfd940e7fca2

### 1. Select

### 2. Poll

### 3. Epoll

#### 3.1 Epoll事件[man]

       The events member is a bit mask composed by ORing together zero or
       more of the following available event types:

       EPOLLIN
              The associated file is available for read(2) operations.

       EPOLLOUT
              The associated file is available for write(2) operations.

       EPOLLRDHUP (since Linux 2.6.17)
              Stream socket peer closed connection, or shut down writing
              half of connection.  (This flag is especially useful for writ‐
              ing simple code to detect peer shutdown when using Edge Trig‐
              gered monitoring.)

       EPOLLPRI
              There is an exceptional condition on the file descriptor.  See
              the discussion of POLLPRI in poll(2).

       EPOLLERR
              Error condition happened on the associated file descriptor.
              This event is also reported for the write end of a pipe when
              the read end has been closed.  epoll_wait(2) will always
              report for this event; it is not necessary to set it in
              events.

       EPOLLHUP
              Hang up happened on the associated file descriptor.
              epoll_wait(2) will always wait for this event; it is not nec‐
              essary to set it in events.

              Note that when reading from a channel such as a pipe or a
              stream socket, this event merely indicates that the peer
              closed its end of the channel.  Subsequent reads from the
              channel will return 0 (end of file) only after all outstanding
              data in the channel has been consumed.

       EPOLLET
              Sets the Edge Triggered behavior for the associated file
              descriptor.  The default behavior for epoll is Level Trig‐
              gered.  See epoll(7) for more detailed information about Edge
              and Level Triggered event distribution architectures.

       EPOLLONESHOT (since Linux 2.6.2)
              Sets the one-shot behavior for the associated file descriptor.
              This means that after an event is pulled out with
              epoll_wait(2) the associated file descriptor is internally
              disabled and no other events will be reported by the epoll
              interface.  The user must call epoll_ctl() with EPOLL_CTL_MOD
              to rearm the file descriptor with a new event mask.

       EPOLLWAKEUP (since Linux 3.5)
              If EPOLLONESHOT and EPOLLET are clear and the process has the
              CAP_BLOCK_SUSPEND capability, ensure that the system does not
              enter "suspend" or "hibernate" while this event is pending or
              being processed.  The event is considered as being "processed"
              from the time when it is returned by a call to epoll_wait(2)
              until the next call to epoll_wait(2) on the same epoll(7) file
              descriptor, the closure of that file descriptor, the removal
              of the event file descriptor with EPOLL_CTL_DEL, or the clear‐
              ing of EPOLLWAKEUP for the event file descriptor with
              EPOLL_CTL_MOD.  See also BUGS.

       EPOLLEXCLUSIVE (since Linux 4.5)
              Sets an exclusive wakeup mode for the epoll file descriptor
              that is being attached to the target file descriptor, fd.
              When a wakeup event occurs and multiple epoll file descriptors
              are attached to the same target file using EPOLLEXCLUSIVE, one
              or more of the epoll file descriptors will receive an event
              with epoll_wait(2).  The default in this scenario (when
              EPOLLEXCLUSIVE is not set) is for all epoll file descriptors
              to receive an event.  EPOLLEXCLUSIVE is thus useful for avoid‐
              ing thundering herd problems in certain scenarios.

              If the same file descriptor is in multiple epoll instances,
              some with the EPOLLEXCLUSIVE flag, and others without, then
              events will be provided to all epoll instances that did not
              specify EPOLLEXCLUSIVE, and at least one of the epoll
              instances that did specify EPOLLEXCLUSIVE.

              The following values may be specified in conjunction with
              EPOLLEXCLUSIVE: EPOLLIN, EPOLLOUT, EPOLLWAKEUP, and EPOLLET.
              EPOLLHUP and EPOLLERR can also be specified, but this is not
              required: as usual, these events are always reported if they
              occur, regardless of whether they are specified in events.
              Attempts to specify other values in events yield the error
              EINVAL.

              EPOLLEXCLUSIVE may be used only in an EPOLL_CTL_ADD operation;
              attempts to employ it with EPOLL_CTL_MOD yield an error.  If
              EPOLLEXCLUSIVE has been set using epoll_ctl(), then a subse‐
              quent EPOLL_CTL_MOD on the same epfd, fd pair yields an error.
              A call to epoll_ctl() that specifies EPOLLEXCLUSIVE in events
              and specifies the target file descriptor fd as an epoll
              instance will likewise fail.  The error in all of these cases
              is EINVAL.
