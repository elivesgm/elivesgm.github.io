---
layout: post
title: "Design Pattern"
date: 2016-10-19 18:15:06 
description: "Design Pattern"
tag: Design Pattern
---

### 1 设计模式基础
- UML, refers to http://www.cnblogs.com/riky/archive/2007/04/07/704298.html
- Tool: EA

#### 1.1 UML
a. 泛化关系(Generalization)

- 用带空心箭头的实线表示。
- 可以用“is a”表示的关系。eg., A cat **is a** (/an) animal。  
- 在C++中表现为继承非抽象类关系, 如下AA继承A，AA is A。

Notes: 抽象类指含有纯虚函数的类

继承非抽象类关系：

	class A {
	public:
		A(){a=0}
		int getA(){return a;}
		~A(){}
	private:
		int a;
	}; //非抽象类
	class AA: public A {
	public:
		AA(){};
	}

b. 实现关系(Realization)

- 用带空心箭头的虚线表示。
- 在C++中表现为继承抽象类关系,如下例中BB继承B。

继承抽象类关系:

	class B {
	public:
		B(){}
		int getB() = 0; //纯虚函数, 只定义接口，没有实现
		~B(){}
	}; //抽象类
	class BB : public B {
	public:
		BB(int value) : bb(value) {}
		int getB(){return bb;}
		~BB(){};
	private:
		int bb;
	};

c. 聚合关系(Aggregation)

- 用一条带空心菱形箭头的直线表示。A--<>B表示A聚合到B，B由A组成。
- 用于表示实体对象之间的关系，表示整体由部分构成，部分不强依赖与整体，部分离开整体依然存在。
- 比如一个部门由员工组成，部门撤销，员工依然存在。

d. 组合关系(Composition)

- 用一条实心菱形的直线表示。A--<=>B表示A组成B，B由A组成。
- 与聚合关系类似，也用于表示实例对象之间的组成关系，但是组合关系的部分与整体之前属于强依赖关系，整体不存在了，部分也不存在了。
- 比如一个公司由部门组成，公司不存在了，部门也就不存在了。

e. 关联关系(Association)

- 用带箭头的实线表示。
- 用于描述不同类的对象之间的结构关系，与运行状态无关，一般用来定义对象之间**静态**的天然的结构，是一种强关联关系。
- 比如学生与学校之间。不强调方向时表示双方互相知道，如果强调方向，如A-->B表示A知道B，B不知道A。如下例子类AA作为类C的成员变量，类C知道类AA的存在，但是类AA不知道类C的存在。

关联关系

	class C {
		AA a;
	};

f. 依赖关系（Dependency)

- 用一条带箭头的虚线表示。
- 用于描述一个对象在运行期间会用到另一个对象的关系。
- 是一种临时性关系，通常在运行期间产生，并且伴随运行时的变化而变化。
- 在代码中体现为类构造方法及类方法的传入参数，箭头指向为调用关系；依赖关系除了临时知道对方外，还使用对方的方法和属性。






### 2 分类

**创建型模式**

- 简单工厂模式( Simple Factory Pattern )
	- 又称为静态工厂方法(Static Factory Method)模式;
	- 根据参数的不同返回不同类的实例;
	- 专门定义一个类来负责创建其他类的实例，被创建的实例通常都具有共同的父类;
	- pro: 简单，无须知道所创建的具体产品类的类名，只需要知道具体产品类所对应的参数即可
	- con: 工厂类集中了所有产品创建逻辑，一旦不能正常工作，整个系统都要受到影响; 添加新产品就不得不修改工厂逻辑; 会增加系统中类的个数

- 工厂方法模式(Factory Method Pattern)
	- 也叫虚拟构造器(Virtual Constructor)模式、多态工厂(Polymorphic Factory)模式；
	- 工厂父类负责定义创建产品对象的公共接口，而工厂子类则负责生成具体的产品对象，这样做的目的是将产品类的实例化操作延迟到工厂子类中完成，即通过工厂子类来确定究竟应该实例化哪一个具体产品类；
	- 核心的工厂类不再负责所有产品的创建，而是将具体创建工作交给子类去做;
	- 在系统中加入新产品时，无须修改抽象工厂和抽象产品提供的接口;
	- pro: 完全符合“开闭原则”;
	- con: 在添加新产品时，需要编写新的具体产品类，而且还要提供与之对应的具体工厂类，系统中类的个数将成对增加，在一定程度上增加了系统的复杂度;

- 抽象工厂模式(Abstract Factory)
	- 提供一个创建一系列相关或相互依赖对象的接口，而无须指定它们具体的类。抽象工厂模式又称为Kit模式;
	- 与工厂方法模式最大的区别在于，工厂方法模式针对的是**一个产品等级结构**，而抽象工厂模式则需要面对**多个产品等级结构**，一个工厂等级结构可以负责多个不同产品等级结构中的产品对象的创建,每个具体的工厂方法可以生成多种具体产品类。
	- pro: 增加新的具体工厂和产品族很方便，无须修改已有系统，符合“开闭原则”;
	- con: 开闭原则的倾斜性（增加新的工厂和产品族容易，增加新的产品等级结构麻烦）
- 建造者模式
	- 又可以称为生成器模式, 将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示;
	- pro:每一个具体建造者都相对独立，而与其他的具体建造者无关，因此可以很方便地替换具体建造者或增加新的具体建造者， 用户使用不同的具体建造者即可得到不同的产品对象; 增加新的具体建造者无须修改原有类库的代码，指挥者类针对抽象建造者类编程，系统扩展方便，符合“开闭原则”。
	- con: 建造者模式所创建的产品一般具有较多的共同点，其组成部分相似，如果产品之间的差异性很大，则不适合使用建造者模式，因此其使用范围受到一定的限制。
- 单例模式
	- 一是某个类只能有一个实例；二是它必须自行创建这个实例；三是它必须自行向整个系统提供这个实例;
	- pro: 可以严格控制客户怎样以及何时访问它，并为设计及开发团队提供了共享的概念; 在系统内存中只存在一个对象，因此可以节约系统资源，对于一些需要频繁创建和销毁的对象，单例模式无疑可以提高系统的性能
	- con: 单例类的职责过重，在一定程度上违背了“单一职责原则”。因为单例类既充当了工厂角色，提供了工厂方法，同时又充当了产品角色

**结构型模式**

1. 适配器模式
2. 桥接模式
3. 装饰模式
4. 外观模式
5. 享元模式
6. 代理模式

**行为型模式**

1. 命令模式
2. 中介者模式
3. 观察者模式
4. 状态模式
5. 策略模式



### 3 设计模式五大原则(SOLID)

- Single Responsibility: every object should only have one reason to change, i.e. every object should perform one thing only.
- Open-Closed: classes should be open for extension and closed for modification.
- Liskov Substitution: you should be able to use a derived class in place of a parent class and it must behave in the same manner.
- Interface Segregation: clients should not be forced to depend on interfaces they do not use.
- Dependency Inversion: helps to decouple your code by ensuring that you depend on abstractions rather than concrete implementations.


References:

1. [Graphic Design Pattern](http://design-patterns.readthedocs.io), readthedocs.io
2. 设计模式六大原则, http://blog.csdn.net/zhengzhb/article/details/7296944
3. 设计模式六大原则, http://www.uml.org.cn/sjms/201211023.asp