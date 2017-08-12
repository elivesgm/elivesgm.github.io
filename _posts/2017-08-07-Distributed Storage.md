---
layout: post
title: "Distributed Storage"
date: 2017-08-07 23:33:06
description: "Tech on Distributed Systems"
tag: Distributed Systems
---


### 1. Introduction

ACID原则是数据库事务正常执行的四个，分别指

- 原子性(Atomicity)，是指一个事务要么全部执行,要么不执行.也就是说一个事务不可能只执行了一半就停止了。
- 一致性(Consistency)，是指事务的运行并不改变数据库中数据的一致性。
- 独立性(Isolation)，事务的独立性也有称作隔离性,是指两个以上的事务不会出现交错执行的状态，因为这样可能会导致数据不一致。
- 持久性(Duration)，指事务执行成功以后,该事务所对数据库所作的更改便是持久的保存在数据库之中，不会无缘无故的回滚。

关系型数据库与非关系型数据库

#### 1.1 SQL

SQL是关系型数据库系统（Relation Database System）的标准语言。所有的关系型数据库管理系统（RDBMS，Relational Database Management System），例如 MySQL、MS Access、Oracle、Sybase、Informix、Postgres SQL和SQL Server，都使用 SQL 作为其标准数据库语言。

SQL包含四个部分：

- 数据定义语言
	- （Data Definition Language，DDL）是SQL语言集中负责数据结构定义与数据库对象定义的语言，由CREATE、ALTER与DROP三个语法所组成
- 数据操纵语言
	- （Data Manipulation Language, DML）是SQL语言中，负责对数据库对象运行数据访问工作的指令集，以INSERT、UPDATE、DELETE三种指令为核心，分别代表插入、更新与删除，是开发以数据为中心的应用程序必定会使用到的指令，因此有很多开发人员都把加上SQL的SELECT语句的四大指令以“CRUD”(create/read/update/delete)来称呼
- 数据控制语言
	-  (Data Control Language) 在SQL语言中，是一种可对数据访问权进行控制的指令，它可以控制特定用户账户对数据表、查看表、预存程序、用户自定义函数等数据库对象的控制权。由 GRANT 和 REVOKE 两个指令组成。
- 事务控制语言


基本概念：

- 什么是表？
	- RDBMS 中的数据存储在被称作表的数据库对象中。表是相互关联的数据记录的集合，由一系列的行和列组成。表是关系型数据库中最常见也是最简单的数据存储形式。
- 什么是字段？
	- 每张表都能够划分成更小的实体——字段。一个字段限定了数据表中的列，被用来维护表中所有记录的特定信息。
- 什么是记录或者数据行？
	- 记录或者说数据行是存在于数据表中的独立条目。记录就是表中水平排列的数据构成的实体。
- 什么是列？
	- 列是表中竖直排列的实体，它包含了表中与某一特定字段相关的所有信息。
- 什么是NULL值？
	- NULL值是表中以空白形式出现的值，表示该记录在此字段处没有设值。NULL值同0值或者包含空格的字段是不同的。值为NULL的字段是在记录创建的时候就被留空的字段。
- SQL 中常见的约束：
	- NOT NULL 约束：保证列中数据不能有 NULL 值
	- DEFAULT 约束：提供该列数据未指定时所采用的默认值
	- UNIQUE 约束：保证列中的所有数据各不相同
	- 主键：唯一标识数据表中的行/记录
	- 外键：唯一标识其他表中的一条行/记录
	- CHECK 约束：此约束保证列中的所有值满足某一条件
	- 索引：用于在数据库中快速创建或检索数据

数据库规范化:

- 数据库规范化指的是对数据库中的数据进行有效组织的过程。对数据库进行规范化主要有两个目的：
	- 消除冗余数据，例如相同数据出现在不同的表中。
	- 保证数据依赖性合理。
- 规范化指导方针分为几种范式（form），你可以把范式想做是数据库的格式或者其结构的布局方式。使用范式的目标是对数据库结构进行整理，从而使其遵循第一范式，接着是第二范式，最终遵循第三范式。
	- 第一范式（1NF）, 第一范式设定了对数据库进行组织的最基本的规范：定义需要的数据项，因为这些项将会成为数据表中的字段。将相关的数据项放在一个表中。
		- 保证不存在重复的数据。
		- 保证有一个主键。
	- 第二范式（2NF）, 第二范式规定，
		- 数据表必须符合第一范式，
		- 并且所有字段与主键之间不存在部分依赖关系。
	- 第三范式（3NF）, 一个数据表符合第三范式，当其满足：
		- 符合第二范式；
		- 所有的非主键字段都依赖于主键；

常用命令

	create database new_dbname;--新建数据库
	drop database old_dbnane; --删除数据库
	show databases;--显示数据库
	use databasename;--使用数据库
	select database();--查看已选择的数据库
	
	show tables;--显示当前库的所有表
	create table tablename(fieldname1 fieldtype1,fieldname2 fieldtype2,..)[ENGINE=engine_name];--创建表
	drop table tablename; --删除表
	create table tablename select statement;--通过子查询创建表
	desc tablename;--查看表结构
	show create table tablename;--查看建表语句
	
	alter table tablename add new_fielname new_fieldtype;--新增列
	alter table tablename add new_fielname new_fieldtype after 列名1;--在列名1后新增列
	alter table tablename modify fieldname new_fieldtype;--修改列
	alter table tablename drop fieldname;--删除列
	alter table tablename_old rename tablename_new;--表重命名
	
	insert into tablename(fieldname1,fieldname2,fieldnamen) valuse(value1,value2,valuen);--增
	delete from tablename [where fieldname=value];--删
	update tablename set fieldname1=new_value where filename2=value;--改
	select * from tablename [where filename=value];--查
	
	truncate table tablename;--清空表中所有数据，DDL语句
	
	show engines;--查看mysql现在已提供的存储引擎:
	show variables like '%storage_engine%';--查看mysql当前默认的存储引擎
	show create table tablename;--查看某张表用的存储引擎（结果的"ENGINE="部分）
	alter table tablename ENGINE=InnoDB--修改引擎
	create table tablename(fieldname1 fieldtype1,fieldname2 fieldtype2,..) ENGINE=engine_name;--创建表时设置存储引擎
	
通过脚本文件操作：

	mysql -D samp_db -u root -p < createtable.sql



#### 1.2 NoSQL

依据结构化方法以及应用场合的不同，主要分为以下几类：

- key-value存储
	- 可以通过key快速查询到其value。一般来说，存储不管value的格式，照单全收。
	- 具有极高的并发读写性能，Redis,MemcacheDB。
- 文档存储
	- 文档存储一般用类似json的格式存储，存储的内容是文档型的。这样也就有有机会对某些字段建立索引，实现关系数据库的某些功能。
	- 可以在海量的数据中快速的查询数据，典型代表为MongoDB以及CouchDB。
- 列存储
	- 顾名思义，是按列存储数据的。最大的特点是方便存储结构化和半结构化数据，方便做数据压缩，对针对某一列或者某几列的查询有非常大的IO优势。Hbase。
- 图存储
	- 图形关系的最佳存储。
- 对象存储
	- 通过类似面向对象语言的语法操作数据库，通过对象的方式存取数据。
- xml存储
	- 高效的存储XML数据，并支持XML的内部查询语法，BaseX。




### 1.3 关系型数据库与非关系型数据库
**关系型数据库**

简单来说，关系模型指的就是二维表格模型，而一个关系型数据库就是由二维表及其之间的联系所组成的一个数据组织。

关系模型中常用的概念：

- 关系：可以理解为一张二维表，每个关系都具有一个关系名，就是通常说的表名
- 元组：可以理解为二维表中的一行，在数据库中经常被称为记录
- 属性：可以理解为二维表中的一列，在数据库中经常被称为字段
- 域：属性的取值范围，也就是数据库中某一列的取值限制
- 关键字：一组可以唯一标识元组的属性，数据库中常称为主键，由一个或多个列组成
- 关系模式：指对关系的描述。其格式为：关系名(属性1，属性2， ... ... ，属性N)，在数据库中成为表结构

关系型数据库的优点：

- 容易理解：二维表结构是非常贴近逻辑世界的一个概念，关系模型相对网状、层次等其他模型来说更容易理解
- 使用方便：通用的SQL语言使得操作关系型数据库非常方便
- 易于维护：丰富的完整性(实体完整性、参照完整性和用户定义的完整性)大大减低了数据冗余和数据不一致的概率

关系型数据库瓶颈：

- 高并发读写需求，网站的用户并发性非常高，往往达到每秒上万次读写请求，对于传统关系型数据库来说，硬盘I/O是一个很大的瓶颈
- 海量数据的高效率读写，网站每天产生的数据量是巨大的，对于关系型数据库来说，在一张包含海量数据的表中查询，效率是非常低的
- 高扩展性和可用性，在基于web的结构当中，数据库是最难进行横向扩展的，当一个应用系统的用户量和访问量与日俱增的时候，数据库却没有办法像web server和app server那样简单的通过添加更多的硬件和服务节点来扩展性能和负载能力。对于很多需要提供24小时不间断服务的网站来说，对数据库系统进行升级和扩展 是非常痛苦的事情，往往需要停机维护和数据迁移。

**NoSQL**
用于指代那些非关系型的，分布式的，且一般不保证遵循ACID原则的数据存储系统


**对比**

- NoSQL数据库仅仅是关系数据库在某些方面（性能，扩展）的一个弥补，单从功能上讲，NoSQL的几乎所有的功能，在关系数据库上都能够满足，所以选择NoSQL的原因并不在功能上。
- 在某些应用场合，比如一些配置的关系键值映射存储、用户名和密码的存储、Session会话存储等等，用NoSQL完全可以替代MySQL存储。不但具有更高的性能，而且开发也更加方便。




### References

[1] [SQL 关系型数据库管理系统](http://wiki.jikexueyuan.com/project/sql/rdbms-concepts.html),jikexueyuan.com.

[2] [Ubuntu下MySQL简单操作](http://www.jianshu.com/p/694d7d0a170b),jianshu.com.