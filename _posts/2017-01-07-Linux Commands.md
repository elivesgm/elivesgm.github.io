---
layout: post
title: "Linux Commands"
date: 2017-01-07 18:15:06 
description: "Useful Linux commands"
tag: Linux
---

### 1. 查看so符号表

在libCavium4J.so中搜索generateKey

>* objdump -tT libCavium4J.so \|grep generateKey
>* nm -D libCavium4J.so  \|grep generateKey
>* grep -iao generateKey libCavium4J.so //其中-i不区分大小写; -a:; -o

### 2. vi/vim常用命令

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

### 3. 查看磁盘使用情况du
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


### 4. 文件操作

a. file, 识别文件类型。

	root@ubuntu:~# file test.o
	test.o: ELF 32-bit LSB relocatable, Intel 80386, version 1 (SYSV), not stripped

b. cat/more/less/nl

- cat, 查看文件所有内容
- more, 和cat的功能一样都是查看文件里的内容，但有所不同的是more可以按页来查看文件的内容，还支持直接跳转行等功能。
- less, 也是对文件或其它输出进行分页显示的工具，应该说是linux正统查看文件内容的工具，功能极其强大。less的用法比起 more 更加的有弹性。 在more的时候，我们并没有办法向前面翻，只能往后面看，但若使用了less时，就可以使用 [pageup] [pagedown] 等按 键的功能来往前往后翻看文件，更容易用来查看一个文件的内容！除此之外，在 less 里头可以拥有更多的搜索功能，不止可以向下搜，也可以向上搜。
- nl, 查看文件的同时显示行号，同cat -n

	root@ubuntu:/home/ubuntu/Codeing/perl# nl MyTestSuit1.h 
	     1	// MyTestSuite1.h

	     2	#include <cxxtest/TestSuite.h>

	     3	class MyTestSuite1: public CxxTest::TestSuite
	     4	{
	     5	public:
	     6	    void testAddition()
	     7	    {
	     8	        TS_ASSERT(1 + 1 > 1);
	     9		TS_ASSERT_EQUALS(1 + 1, 2);
	    10	    }
	    11	};
	root@ubuntu:/home/ubuntu/Codeing/perl# cat -n  MyTestSuit1.h 
	     1	// MyTestSuite1.h
	     2	
	     3	#include <cxxtest/TestSuite.h>
	     4	
	     5	class MyTestSuite1: public CxxTest::TestSuite
	     6	{
	     7	public:
	     8	    void testAddition()
	     9	    {
	    10	        TS_ASSERT(1 + 1 > 1);
	    11		TS_ASSERT_EQUALS(1 + 1, 2);
	    12	    }
	    13	};


c. tee, 输出到屏幕同时重定向到文件。

	root@ubuntu:/home/ubuntu/Codeing/perl# cat test.perl | tee logfile
	#!/usr/bin/perl

	@array = ("ACTC", "AATT", "GTCA");

	foreach $para (@array){
	    @data = split("", $para);
	    print "@data[1]\n";
	}

d. od, object dump, 以指定格式显示文件。

	root@ubuntu:/home/ubuntu/Codeing/perl# od test.perl 
	0000000 020443 072457 071163 061057 067151 070057 071145 005154
	0000020 040012 071141 060562 020171 020075 021050 041501 041524
	0000040 026042 021040 040501 052124 026042 021040 052107 040503
	0000060 024442 005073 063012 071157 060545 064143 022040 060560
	0000100 060562 024040 060500 071162 074541 075451 020012 020040
	0000120 040040 060544 060564 036440 071440 066160 072151 021050
	0000140 026042 022040 060560 060562 035451 020012 020040 070040
	0000160 064562 072156 021040 062100 072141 055541 056461 067134
	0000200 035442 076412 000012
	0000205
	root@ubuntu:/home/ubuntu/Codeing/perl# od -c test.perl 
	0000000   #   !   /   u   s   r   /   b   i   n   /   p   e   r   l  \n
	0000020  \n   @   a   r   r   a   y       =       (   "   A   C   T   C
	0000040   "   ,       "   A   A   T   T   "   ,       "   G   T   C   A
	0000060   "   )   ;  \n  \n   f   o   r   e   a   c   h       $   p   a
	0000100   r   a       (   @   a   r   r   a   y   )   {  \n            
	0000120       @   d   a   t   a       =       s   p   l   i   t   (   "
	0000140   "   ,       $   p   a   r   a   )   ;  \n                   p
	0000160   r   i   n   t       "   @   d   a   t   a   [   1   ]   \   n
	0000200   "   ;  \n   }  \n
	0000205

e. head/tail

	root@ubuntu:/home/ubuntu/Codeing/perl# head -n 3 MyTestSuit1.h
	// MyTestSuite1.h

	#include <cxxtest/TestSuite.h>
	root@ubuntu:/home/ubuntu/Codeing/perl# tail -n 3 MyTestSuit1.h
		TS_ASSERT_EQUALS(1 + 1, 2);
	    }
	};

f. grep
在logfile.txt中搜索memory并将其前10行到后5行重定向到tmp.txt:

	grep "memory" -n -A10 -B5 logfile.txt > tmp.txt



### 5. user/group

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


### 6. CLI命令

a. ctrl+x: 行首行尾跳转

b. tree(需要额外安装):

	root@ubuntu:/home/ubuntu/code# tree
	.
	├── dev_test
	│   └── core
	└── git_repo
	
	2 directories, 1 file


### 7. sed
依照script的指令，来处理、编辑文本文件。
语法:
	
	sed [-hnV][-e<script>][-f<script文件>][文本文件]

参数说明：

- -e<script\>或--expression=<script\> 以选项中指定的script来处理输入的文本文件。
- -f<script文件\>或--file=<script文件\> 以选项中指定的script文件来处理输入的文本文件。
- -h或--help 显示帮助。
- -n或--quiet或--silent 仅显示script处理后的结果。
- -V或--version 显示版本信息。

动作说明：

- a ：新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)～
- c ：取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
- d ：删除，因为是删除啊，所以 d 后面通常不接任何咚咚；
- i ：插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
- p ：列印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行～
- s ：取代，可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正规表示法！例如 1,20s/old/new/g 就是啦！

Example:
在testfile文件的第四行后**添加(a)**一行，并将结果输出到标准输出:

	root@ubuntu:/home/ubuntu/code# sed -e 4a\newLine testfile 

将第2-5行的内容**取代(c)**成为"No 2-5 number":

	root@ubuntu:/home/ubuntu/code# nl /etc/passwd | sed '2,5c No 2-5 number'
	1 root:x:0:0:root:/root:/bin/bash
	No 2-5 number
	6 sync:x:5:0:sync:/sbin:/bin/sync

**删除(d)**第 3 到最后一行:

	root@ubuntu:/home/ubuntu/code# nl /etc/passwd | sed '3,$d'

test.txt最后一行($)**插入(i)**新增(a)内容"This is a test":

	root@ubuntu:/home/ubuntu/code# sed -i '$a This is a test' test.txt

**列出(p)**/etc/passwd 文件内的第 5-7 行:

	root@ubuntu:/home/ubuntu/code# nl /etc/passwd | sed -n '5,7p'
	5 lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
	6 sync:x:5:0:sync:/sbin:/bin/sync
	7 shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown

用行为单位进行部分数据的搜寻并**取代(s)**:

	sed 's/要被取代的字串/新的字串/g'

将 IP 前面的部分予以删除:

	root@ubuntu:/home/ubuntu/code# /sbin/ifconfig eth0
	eth0 Link encap:Ethernet HWaddr 00:90:CC:A6:34:84
	inet addr:192.168.1.100 Bcast:192.168.1.255 Mask:255.255.255.0
	
	root@ubuntu:/home/ubuntu/code# /sbin/ifconfig eth0 | grep 'inet addr' | sed 's/^.*addr://g'
    192.168.1.100 Bcast:192.168.1.255 Mask:255.255.255.0


### 8. awk

a. 语法:

	awk [选项参数] 'script' var=value file(s)
	或
	awk [选项参数] -f scriptfile var=value file(s)

选项参数说明：

- -F fs or --field-separator fs, 指定输入文件折分隔符，fs是一个字符串或者是一个正则表达式，如-F:。
- -v var=value or --asign var=value, 赋值一个用户定义变量。
- -f scripfile or --file scriptfile, 从脚本文件中读取awk命令。

b. 基本用法：

	awk '{[pattern] action}' {filenames}   # 行匹配语句 awk '' 只能用单引号
    awk -F  #-F相当于内置变量FS, 指定分割字符

c. Example:

每行按空格或TAB分割，输出文本中的1、4项：

	root@ubuntu:/home/ubuntu/code# cat test.txt
	2 this is a test
	3 Are you like awk
	root@ubuntu:/home/ubuntu/code# awk '{print $1,$4}' test.txt
	 2 a
	 3 like

格式化输出：

	root@ubuntu:/home/ubuntu/code# awk '{printf "%-8s %-10s\n",$1,$4}' test.txt
	 2        a
	 3        like

使用","分割：

    root@ubuntu:/home/ubuntu/code# awk -F, '{print $1,$2}'   test.txt
	 2 this is a test
	 3 Are you like awk

过滤第一列大于2的行：

	root@ubuntu:/home/ubuntu/code# awk '$1>2' test.txt



### 9. 其他

- Kernel code: http://home.ustc.edu.cn/~boj/courses/linux_kernel/1_boot.html
- Valgrind: http://blog.csdn.net/sduliulun/article/details/7732906



### References:

[1] [linux-command-manual](http://www.runoob.com/linux/linux-command-manual.html), runoob.com.
