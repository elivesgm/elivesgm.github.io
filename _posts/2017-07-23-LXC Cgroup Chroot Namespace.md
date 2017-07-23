---
layout: post
title: "LXC Chroot Cgroup Namespace"
date: 2017-07-23 17:20:00 
description: "LXC cgroup chroot namespace"
tag: Linux
---

### 0 LXC
LXC, LinuX Containers，它是一个加强版的Chroot。简单的说，LXC就是将不同的应用隔离开来，这有点类似于chroot，chroot是将应用隔离到一个虚拟的私有root下，而LXC在这之上更进了一步。LXC内部依赖Linux内核的3种隔离机制（isolation infrastructure）：

- Chroot
- Cgroups
- Namespaces


### 1. chroot
chroot提供了一种简单的隔离模式：chroot内部的文件系统无法访问外部的内容.

#### 1.1 简介[2]
chroot，即 change root directory (更改 root 目录)。在 linux 系统中，系统默认的目录结构都是以 `/`，即是以根 (root) 开始的。而在使用 chroot 之后，系统的目录结构将以指定的位置作为 `/` 位置。

#### 1.2 使用chroot的原因[2]
在经过 chroot 之后，系统读取到的目录和文件将不在是旧系统根下的而是新根下(即被指定的新的位置)的目录结构和文件，因此它带来的好处大致有以下3个：

- 增加了系统的安全性，限制了用户的权力；
	- 在经过 chroot 之后，在新根下将访问不到旧系统的根目录结构和文件，这样就增强了系统的安全性。这个一般是在登录 (login) 前使用 chroot，以此达到用户不能访问一些特定的文件。
- 建立一个与原系统隔离的系统目录结构，方便用户的开发；
	- 使用 chroot 后，系统读取的是新根下的目录和文件，这是一个与原系统根下文件不相关的目录结构。在这个新的环境中，可以用来测试软件的静态编译以及一些与系统不相关的独立开发。
- 切换系统的根目录位置，引导 Linux 系统启动以及急救系统等。
	- chroot 的作用就是切换系统的根位置，而这个作用最为明显的是在系统初始引导磁盘的处理过程中使用，从初始 RAM 磁盘 (initrd) 切换系统的根位置并执行真正的 init。另外，当系统出现一些问题时，我们也可以使用 chroot 来切换到一个临时的系统。



### 2. cgroup[3]
#### 2.1 简介
CGroup 是 Control Groups 的缩写，是 Linux 内核提供的一种可以限制、记录、隔离进程组 (process groups) 所使用的物力资源 (如 cpu memory i/o 等等) 的机制。2007 年进入 Linux 2.6.24 内核，CGroups 不是全新创造的，它将进程管理从 cpuset 中剥离出来，作者是 Google 的 Paul Menage。CGroups 也是 LXC 为实现虚拟化所使用的资源管理手段。

CGroup 是将任意进程进行分组化管理的 Linux 内核功能。CGroup 本身是提供将进程进行分组化管理的功能和接口的基础结构，I/O 或内存的分配控制等具体的资源管理功能是通过这个功能来实现的。这些具体的资源管理功能称为 CGroup 子系统或控制器。CGroup 子系统有控制内存的 Memory 控制器、控制进程调度的 CPU 控制器等。

#### 2.2 概念
CGroup 相关概念解释

- 任务（task）。在 cgroups 中，任务就是系统的一个进程；
- 控制族群（control group）。控制族群就是一组按照某种标准划分的进程。Cgroups 中的资源控制都是以控制族群为单位实现。一个进程可以加入到某个控制族群，也从一个进程组迁移到另一个控制族群。一个进程组的进程可以使用 cgroups 以控制族群为单位分配的资源，同时受到 cgroups 以控制族群为单位设定的限制；
- 层级（hierarchy）。控制族群可以组织成 hierarchical 的形式，既一颗控制族群树。控制族群树上的子节点控制族群是父节点控制族群的孩子，继承父控制族群的特定的属性；
- 子系统（subsystem）。一个子系统就是一个资源控制器，比如 cpu 子系统就是控制 cpu 时间分配的一个控制器。子系统必须附加（attach）到一个层级上才能起作用，一个子系统附加到某个层级以后，这个层级上的所有控制族群都受到这个子系统的控制。



### 3. namespace[3,4]
Linux Namespace在chroot基础上，提供了对UTS、IPC、mount、PID、network、User等的隔离机制

Linux的3.12内核支持6种Namespace：

- UTS: hostname
- IPC: 进程间通信
- PID: "chroot"进程树
- NS: 挂载点，首次登陆Linux
- NET: 网络访问，包括接口
- USER: 将本地的虚拟user-id映射到真实的user-id（之后的文章会讲到）

具体参见[5].


### References

[1] [DOCKER基础技术：LINUX NAMESPACE](http://coolshell.cn/articles/17010.html), coolshell.cn.

[2] [理解 chroot](https://www.ibm.com/developerworks/cn/linux/l-cn-chroot/index.html), ibm.com.

[3] [CGroup 介绍、应用实例及原理描述](https://www.ibm.com/developerworks/cn/linux/1506_cgroup/index.html), ibm.com.

[4] [Linux命名空间学习教程](http://dockone.io/article/76), dockone.io.