---
layout: post
title: "CxxTest with MockCpp"
date: 2017-05-17 16:20:06 
description: "Development Test"
tag: Test
---


### 1. CxxTest

#### A Simple Example
----------------

- Create a test suite header file:

        //file name : MyTestSuite.h
        #include <cxxtest/TestSuite.h>
        class MyTestSuite : public CxxTest::TestSuite 
        {
         public:

             void testAddition( void )
             {
             TS_ASSERT( 1 + 1 > 1 );
             TS_ASSERT_EQUALS( 1 + 1, 2 );
             }
         };

- Generate the tests file:

>* \# cxxtestgen --error-printer -o tests.cpp MyTestSuite.h

- Compile and run:

>* \# g++ -o main tests.cpp
>* \# ./main

- Result:
>* Running cxxtest tests (1 test).OK!

### 2. MockCpp
#### 2.1 mock++当前支持七种类型的约束/行为：

- 调用次数约束 —— expects()/stubs()
- 调用者选择器 —— caller()
- 先行调用约束 —— before()
- 调用参数约束 —— with()
- 后行调用约束 —— after()
- 函数调用行为 —— will()/then()
- 标识符指定 —— id()

除了调用次数约束之外，所有其它的调用约束或行为都可以任意，eg：

    MOCK_METHOD(mock, method)
       .expects(once())
       .before(anotherMock,"close")
       .with(eq(1), any(), neq(2.0))
       .after(anotherMock,"open")
       .will(returnValue(true))
       .then(throws(std::exception))
       .id("myMethod");

#### 2.2 函数调用行为
函数调用行为是mock机制中最重要的部分。也是mock存在最基本的价值所在。通过指定函数调用行为，mock更好得实现了stub的功能。
行为类型

- 返回一个值： returnValue (value)
- 返回一系列的值：returnObjectList (o1, o2,…)
- 抛出异常： throws (exception)
- 忽略返回值： ignoreReturnValue ()
- 转调一个stub函数：invoke(stubFunction)
- 重复返回一个值： repeat(value, times)
- 步增一个值：increase(from, to)/increase(from)

函数调用行为通过will(behavior)/then(behavior)来指定。then()必须在will()之后。will()只会出现一次，但then()可以出现任意多次。比如：

    MOCK_METHOD(mock, foo)
        .stubs()
        .will(returnValue(10))
        .then(repeat(20,2))
        .then(throws(std::exception))
        .then(returnValue(5));

当你使用will()/then()来指定函数调用行为时，指定的函数调用行为将会按照指定的顺序依次发生作用，**如果指定的最后一个函数调用行为发生了作用之后，仍然有进一步的调用发生，如果其它约束条件都满足，则最后一个函数调用行为将持续发生作用**。比如上一个例子将会产生如下结果：

    TS_ASSERT_EQUALS(10, mock->foo());
    TS_ASSERT_EQUALS(20, mock->foo());
    TS_ASSERT_EQUALS(20, mock->foo());
    TS_ASSERT_THROWS(mock->foo(), std::exception);
    TS_ASSERT_EQUALS(5, mock->foo());
    TS_ASSERT_EQUALS(5, mock->foo());//特别是需要注意在一个testcase中mock的函数调用行为在后面的testcase中如果约束满足也会继续发生作用
    TS_ASSERT_EQUALS(5, mock->foo());//特别是需要注意在一个testcase中mock的函数调用行为在后面的testcase中如果约束满足也会继续发生作用

#### 2.3 调用参数约束
调用参数事实上是一种选择器，而不是一种约束。因为你可以在一个用例中，针对一个函数，指定多个mock调用方式。当被测函数调用这个mock函数时，mock++会找到第一个匹配所有选择器的调用方式，然后返回这个调用方式指定的结果（返回值，抛出异常，等等）。如果mock++找不到任何匹配的调用方式，则会让这个用例失败。约束的类型：

**输入参数约束**：

- 相等： eq (value)
- 不等： neq (value)
- 大于 : gt (value)
- 小于 : lt (vlaue)
- 内存匹配:
    mirror (object);
    mirror (address, size)
- 对象内容监控: spy(address, size)
- 引用传值: outBound(object)
- 指针传值: outBoundP(address)
- 字符串匹配：
    smirror (str)
    startWith (str)
    endWith (str)
    contains (str)
- 占位符：any ()

**输出参数约束**：
- 通过引用得到输出参数：outBound (reference, 输入参数约束)
- 通过指针得到输出参数：outBoundP (pointer, 输入参数约束)

调用参数约束通过with(contraint1, contraint2, …)指定。如果想指定任何一个参数约束，则必须使用with()。在with()里，参数约束必须按照函数定义的参数顺序指定。对于不需要指定约束的参数，可以通过any() 来忽略约束；最后一个有效约束后面的any()可以省略。

其中，mirror()用来约束内存内容的严格相等性。如果用来约束对象内容，则可以忽略size参数；如果用来约束非单一对象的内容，则需要通过size参数以字节为单位来指定内存的大小。需要注意的是，当约束单一对象内容时，由于对齐问题所造成的padding字节内容的随机性，这种约束可能会带来错误的结果。对于C程序员，可以通过memset函数来清零内存。对于C++程序员，则可以通过重载对象的双等(==)操作符，然后通过eq()来指定相应的约束。

startWith()，endWith(), contains()用来约束字符串的存在性，startWith()约束参数是否以指定的字符串开始，endWith()约束参数是否以指定的字符串结束；而contains()则不关心指定的字符串在参数的任何位置，只关心它的存在。

#### 2.4 调用次数约束
调用次数约束通过expects(times)指定。其中times是约束的类型，比如once()；如果不关心调用次数，则使用stubs()。
**约束的类型**

- 一次：once ()
- 准确的次数：exactly (n)
- 至少：
    -atLeast (n)
    -atLeastOnce()
- 至多：
   -atMost (n)
   -atMostOnce()
- 不调用：never ()

#### 2.5 调用顺序

**约束的类型**

- 之前：
   -before (id)
   -before (object, id)
- 之后：
   -after (id)
   -after (object, id)

Eg.,

    MockObject<Interface> mock;

    MOCK_METHOD(mock, foo)
                .stubs()
                .will(returnValue(10))
                .id("foo");

    MOCK_METHOD(mock, bar)
                .stubs()
                .will(returnValue(true))
                .id("bar");

    MOCK_METHOD(mock, fix)
                .stubs()
                .after("foo")
                .after("bar")
                .will(returnValue(true));

#### 2.6 setup/teardown

在teardown()中调用GlobalMockObject::verify(),否则会调用根据约束满足的第一个mock。


#### 2.7 使用注意事项

* mockcpp是强类型检查的，强类型检查也是C/C＋＋的一个优势，比如eq(3)，如果调用函数时用的参数是UL，那么就应该用eq((UL)3)
* mock方法
- C函数或者类的静态成员方法用MOCKER;
- mock 类的非静态成员方法需要先用MockObject\<MyClass\>mocker声明一个mock对象，再用MOCK_METHOD(mocker, method)来mock指定方法。
* MOCKER/MOCK_METHOD之后是stubs、defaults、expects，三个必须有一个。
* 使用mockcpp时，校验是否按照mock规范进行调用的，应该用： GlobalMockObject::verify(); verify之后，会自动执行reset（如果是对象的mock，应该用mocker.verify()，同样也会自动reset。）。如果单单只想reset，也可以直接使用GlobalMockObject::reset(), 但是不推荐这样;一般是在teardown中调用verify。

