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


### 5 其他

- Kernel code: http://home.ustc.edu.cn/~boj/courses/linux_kernel/1_boot.html
- Core Dump: http://www.cnblogs.com/hazir/p/linxu_core_dump.html
- Valgrind: http://blog.csdn.net/sduliulun/article/details/7732906
