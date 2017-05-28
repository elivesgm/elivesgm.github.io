---
layout: post
title: "Linux Commands"
date: 2017-01-07 18:15:06 
description: "Useful Linux commands"
tag: Linux
---

### 1 查看so符号表

在libCavium4J.so中搜索generateKey

>* objdump -tT libCavium4J.so \|grep generateKey
>* nm -D libCavium4J.so  \|grep generateKey
>* grep -iao generateKey libCavium4J.so //其中-i不区分大小写; -a:; -o

### 2 vi/vim常用命令

- /abc, 从光标开始处向文件尾搜索abc, 之后n/N分别表示跳转到下一个/上一个搜索结果；
- ?abc, 从光标开始处向文件首搜索abc；
- n, 在同一方向重复上一次搜索命令 
- N, 在反方向上重复上一次搜索命令 
- :set nu, 显示行号
- :set nonu, 不显示行号
- :234, 跳转到234行
- h Move left
- j Move down
- k Move up
- l Move right
- w Move to next word
- W Move to next blank delimited word
- b Move to the beginning of the word
- B Move to the beginning of blank delimted word
- e Move to the end of the word
- E Move to the end of Blank delimited word
- ( Move a sentence back
- ) Move a sentence forward
- { Move a paragraph back
- } Move a paragraph forward
- 0 Move to the begining of the line
- $ Move to the end of the line
- 1G Move to the first line of the file
- G Move to the last line of the file
- nG Move to nth line of the file
- :n Move to nth line of the file
- fc Move forward to c
- Fc Move back to c
- H Move to top of screen
- M Move to middle of screen
- L Move to botton of screen
- % Move to associated ( ), { }, [ ]

### 3 查看磁盘使用情况du
#### 3.1 常用选项
- -a：显示目录占用的磁盘空间大小，还要显示其下目录和文件占用磁盘空间的大小
- -c：显示几个目录或文件占用的磁盘空间大小，还要统计它们的总和
- -h：以人类可读的方式显示
- -l ：统计硬链接占用磁盘空间的大小
- -L：统计符号链接所指向的文件占用的磁盘空间大小
- -s：显示目录占用的磁盘空间大小，不要显示其下子目录和文件占用的磁盘空间大小
- --apparent-size：显示目录或文件自身的大小
- --max-depth：目录深度

#### 3.2 常用命令
- du -ah, 查看目录下所有文件和文件夹（包含子目录）所占用的磁盘空间大小; du -ah --max-depth=1查看深度为1的所有文件和文件夹所占用的磁盘空间大小
- du -sh, 查看目录或文件的所占用的磁盘空间大小。


### 4 识别文件类型file

	root@ubuntu:~# file test.o
	test.o: ELF 32-bit LSB relocatable, Intel 80386, version 1 (SYSV), not stripped


### 5 cat/more/less

- cat, 查看文件所有内容
- more, 和cat的功能一样都是查看文件里的内容，但有所不同的是more可以按页来查看文件的内容，还支持直接跳转行等功能。
- less, 也是对文件或其它输出进行分页显示的工具，应该说是linux正统查看文件内容的工具，功能极其强大。less的用法比起 more 更加的有弹性。 在more的时候，我们并没有办法向前面翻，只能往后面看，但若使用了less时，就可以使用 [pageup] [pagedown] 等按 键的功能来往前往后翻看文件，更容易用来查看一个文件的内容！除此之外，在 less 里头可以拥有更多的搜索功能，不止可以向下搜，也可以向上搜。

### 6 user/group

a. 查看系统中user/group信息

	root@ubuntu:~# cat /etc/passwd
	ubuntu:x:1000:1000:ubuntu,,,:/home/ubuntu:/bin/bash

b. w命令用于显示已经登录系统的用户的名称，以及他们正在做的事。该命令所使用的信息来源于/var/run/utmp文件。

	root@ubuntu:~# w
	 00:14:33 up  2:36,  3 users,  load average: 0.00, 0.00, 0.01
	USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
	ubuntu   tty7     :0               21:43    2:36m  3:04   0.27s /sbin/upstart --user
	root     tty8     :3               23:07    2:36m  3.15s  0.16s /sbin/upstart --user
	root     pts/19   192.168.1.101    00:13    0.00s  0.03s  0.00s w

c. who命令查看（登录）用户名称及所启动的进程。

	root@ubuntu:~# who
	ubuntu   tty7         2017-05-25 21:43 (:0)
	root     tty8         2017-05-25 23:07 (:3)
	root     pts/19       2017-05-26 00:13 (192.168.1.101)

d. users命令，可用于打印输出登录服务器的用户名称。

	root@ubuntu:~# users
	root root ubuntu

e. whoami命令查看当前所使用的登录名称

	root@ubuntu:~# whoami
	root

f. last命令可用于显示特定用户登录系统的历史记录。

	root@ubuntu:~# last root
	root     pts/19       192.168.1.101    Fri May 26 00:13   still logged in
	root     tty8         :3               Thu May 25 23:07    gone - no logout
	root     pts/18       192.168.1.101    Thu May 25 22:44 - 23:45  (01:01)


### 7 CLI命令

ctrl+x: 行首行尾跳转

### 7 其他

- Kernel code: http://home.ustc.edu.cn/~boj/courses/linux_kernel/1_boot.html
- Core Dump: http://www.cnblogs.com/hazir/p/linxu_core_dump.html
- Valgrind: http://blog.csdn.net/sduliulun/article/details/7732906
