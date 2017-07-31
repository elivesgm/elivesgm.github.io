---
layout: post
title: "Programming Frameworks"
date: 2016-09-24 14:15:06 
description: "Programming Skills"
tag: Programming Skills
---


### 1. 事件驱动与消息驱动

事件模式耦合高，同模块内好用；消息模式耦合低，跨模块好用。事件模式集成其它语言比较繁琐，消息模式集成其他语言比较轻松。事件是侵入式设计，霸占你的主循环；消息是非侵入式设计，将主循环该怎样设计的自由留给用户。如果你在设计一个东西举棋不定，那么你可以参考win32的GetMessage，本身就是一个藕合度极低的接口，又足够自由，接口任何语言都很方便，具体应用场景再在其基础上封装成事件并不是难事，接口耦合较低，即便哪天事件框架调整，修改外层即可，不会伤经动骨。而如果直接实现成事件，那就完全反过来了。




### Reference

[1] [事件驱动和消息驱动](http://www.jianshu.com/p/df711376c33e), jianshu.com.