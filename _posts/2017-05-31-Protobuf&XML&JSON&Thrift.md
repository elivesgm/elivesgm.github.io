---
layout: post
title: "Protobuf vs XML vs JSON vs Thrift"
date: 2017-05-31 18:15:06 
description: "结构化数据存储格式"
tag: RPC   
---

### 1. [Protobuf](https://developers.google.com/protocol-buffers/)

#### 1.1 Introduction

>Protocol buffers are Google's language-neutral, platform-neutral, extensible mechanism for serializing structured data – think XML, but smaller, faster, and simpler. You define how you want your data to be structured once, then you can use special generated source code to easily write and read your structured data to and from a variety of data streams and using a variety of languages[1].

Google Protocol Buffer( 简称 Protobuf) 是 Google 公司内部的混合语言数据标准，目前已经正在使用的有超过 48,162 种报文格式定义和超过 12,183 个 .proto 文件。他们用于 RPC 系统和持续数据存储系统[2]。

#### 1.2 Usages

#### 1.3 比较[2]

- Protobuf的优点
	- Protobuf 有如 XML，不过它更小、更快、也更简单。你可以定义自己的数据结构，然后使用代码生成器生成的代码来读写这个数据结构。你甚至可以在无需重新部署程序的情况下更新数据结构。只需使用 Protobuf 对数据结构进行一次描述，即可利用各种不同语言或从各种不同数据流中对你的结构化数据轻松读写。
	- “向后”兼容性好，人们不必破坏已部署的、依靠“老”数据格式的程序就可以对数据结构进行升级。
	- 语义更清晰，无需类似 XML 解析器的东西（因为 Protobuf 编译器会将 .proto 文件编译生成对应的数据访问类以对 Protobuf 数据进行序列化、反序列化操作）。
	- 使用 Protobuf 无需学习复杂的文档对象模型，Protobuf 的编程模式比较友好，简单易学，同时它拥有良好的文档和示例，对于喜欢简单事物的人们而言，Protobuf 比其他的技术更加有吸引力。

- Protobuf 的不足
	- Protbuf 与 XML 相比也有不足之处。它功能简单，无法用来表示复杂的概念。
	- 由于文本并不适合用来描述数据结构，所以 Protobuf 也不适合用来对基于文本的标记文档（如 HTML）建模。另外，由于 XML 具有某种程度上的自解释性，它可以被人直接读取编辑，在这一点上 Protobuf 不行，它以二进制的方式存储，除非你有 .proto 定义，否则你没法直接读出 Protobuf 的任何内容

#### 1.4 原理

同XML相比，Protobuf的主要优点在于性能高。它以高效的二进制方式存储，比XML小3到10倍，快20 到100 倍。

- 第一点，Protobuf采用Varint编码方式，信息的表示非常紧凑，这意味着消息的体积减少，自然需要更少的资源。比如网络上传输的字节数更少，需要的IO更少等，从而提高性能。
- 第二点，相较于XML，Protobuf只需要简单地将一个二进制序列，按照指定的格式读取到C++对应的结构类型中就可以了。

##### 1.4.1 采用Varint编码方式

Varint 是一种紧凑的表示数字的方法。它用一个或多个字节来表示一个数字，值越小的数字使用越少的字节数。这能减少用来表示数字的字节数。Varint 中的每个 byte 的最高位bit有特殊的含义，如果该位为1，表示后续的byte也是该数字的一部分，如果该位为 0，则结束。其他的7个bit都用来表示数字。因此小于128的数字都可以用一个byte表示。大于128的数字，比如300，会用两个字节来表示：1010 1100 0000 0010。

消息经过序列化后会成为一个二进制数据流，该流中的数据为一系列的Key-Value对。采用这种 Key-Pair 结构无需使用分隔符来分割不同的 Field。对于可选的 Field，如果消息中不存在该 field，那么在最终的 Message Buffer 中就没有该 field，这些特性都有助于节约消息本身的大小。假设我们生成如下的一个消息Test1:

	Test1.id = 10; 
	Test1.str = “hello”；

则最终的 Message Buffer 中有两个 Key-Value 对，一个对应消息中的 id；另一个对应 str。其中，Key的定义如下：
 (field_number << 3) | wire_type
可以看到 Key 由两部分组成。第一部分是 field_number，比如消息 lm.helloworld 中 field id 的 field_number 为 1。第二部分为 wire_type。表示 Value 的传输类型。

消息Test1通过Protobuf序列化后的字节序列为：

	08 65 12 06 48 65 6C 6C 6F 77

而如果用 XML，对应ASCII码表示和序列化表示如下（一共55个字节）：

	<helloworld> 
	   <id>101</id> 
	   <name>hello</name> 
	</helloworld>

	31 30 31 3C 2F 69 64 3E 3C 6E 61 6D 65 3E 68 65 
	6C 6C 6F 3C 2F 6E 61 6D 65 3E 3C 2F 68 65 6C 6C 
	6F 77 6F 72 6C 64 3E 

可以看出Protobuf消息的内容小，适于网络传输。

##### 1.4.2 封解包的方式

XML的封解包过程：

- 从文件中读取出字符串，再转换为 XML 文档对象结构模型
- 之后，再从XML文档对象结构模型中读取指定节点的字符串
- 最后再将这个字符串转换成指定类型的变量。

这个过程非常复杂，其中将XML文件转换为文档对象结构模型的过程通常需要完成词法文法分析等大量消耗CPU的复杂计算。

Protobuf只需要简单地将一个二进制序列，按照指定的格式读取到C++对应的结构类型中就可以了。从上一节的描述可以看到消息的 decoding 过程也可以通过几个位移操作组成的表达式计算即可完成。速度非常快。




### 2. XML
XML 指可扩展标记语言（EXtensible Markup Language）

- 设计宗旨是传输数据，而非显示数据
- 标签没有被预定义, 需要自行定义标签。
- 被设计为具有自我描述性。
- XML文档形成了一种树结构，它从“根部”开始，然后扩展到“枝叶”。
- 在XML中，省略关闭标签是非法的。所有元素都必须有关闭标签



### 3. JSON
JSON 指的是 JavaScript 对象表示法（JavaScript Object Notation）

- 是轻量级的文本数据交换格式
- 独立于语言
- 具有自我描述性，更易理解

JSON 对象在花括号中书写,对象可以包含多个名称/值对：
	
	{ "firstName":"John" , "lastName":"Doe" }


### 4. Thrift


### 5. 对比
#### 5.1 XML
- 优点
	- 格式统一，符合标准；
	- 容易与其他系统进行远程交互，数据共享比较方便。
- 缺点
	- XML文件庞大，文件格式复杂，传输占带宽；
	- 服务器端和客户端都需要花费大量代码来解析XML，导致服务器端和客户端代码变得异常复杂且不易维护；
	- 客户端不同浏览器之间解析XML的方式不一致，解析得考虑子节点父节点，比较困难；
	- 服务器端和客户端解析XML花费较多的资源和时间。

#### 5.2 JSON
- 优点：
	- 数据格式比较简单，易于读写，格式都是压缩的，相对于XML来讲，数据的体积小，传递的速度更快些；
	- 易于解析，客户端JavaScript可以简单的通过eval()进行JSON数据的读取；
	- 因为JSON格式能直接为服务器端代码使用，大大简化了服务器端和客户端的代码开发量，且完成任务不变，并且易于维护。
- 缺点
	- 没有XML格式这么推广的深入人心和喜用广泛，描述性比XML较差，没有XML那么通用性；

JSON解析/生成库，refers to [5]

#### 5.3 Example:

XML:

	<?xml version="1.0" encoding="utf-8" ?>
	<country>
	  <name>中国</name>
	  <province>
	    <name>黑龙江</name>
	    <citys>
	      <city>哈尔滨</city>
	      <city>大庆</city>
	    </citys>  　　
	  </province>
	  <province>
	    <name>广东</name>
	    <citys>
	      <city>广州</city>
	      <city>深圳</city>
	      <city>珠海</city>
	    </citys> 　　
	  </province>
	</country>

JSON格式：

	var country =
	        {
	            name: "中国",
	            provinces: [
	            { name: "黑龙江", citys: { city: ["哈尔滨", "大庆"]} },
	            { name: "广东", citys: { city: ["广州", "深圳", "珠海"]} }
	            ]
	        }

### References

[1] [protocol-buffers](https://developers.google.com/protocol-buffers/), google developers.

[2] [Google Protocol Buffer的使用和原理](https://www.ibm.com/developerworks/cn/linux/l-cn-gpb/), IBM developeWorks.

[3] [google protobuf安装与使用](http://www.cnblogs.com/luoxn28/p/5303517.html), cnblogs.com.

[4] [protobuf-c](http://www.cnblogs.com/Anker/p/3416541.html),cnblogs.com.

[5] [nativejson-benchmark](https://github.com/miloyip/nativejson-benchmark), github.com
