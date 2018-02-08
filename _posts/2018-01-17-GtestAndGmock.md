---
layout: post
title: "gtest and gmock"
date: 2018-01-17 13:20:06 
description: "Development Test"
tag: Test
---



### Gtest

1. 下载安装

    tar -xvf googletest-release-1.8.0.tar
    cd test/gtest/googletest-release-1.8.0/googletest/
    cmake CMakeLists.txt
    make

2. 准备头文件和lib库文件

  cp -r include/ ~/test/gtest/
  cp lib* ~/test/gtest/lib/

3. 待测试源码

    [^_^ 11:52 ~/test/gtest/testcase] vim sample.h
    // sample.h
    int fun(int a, int b);

    [^_^ 11:52 ~/test/gtest/testcase] vim sample.cpp
    include "sample.h"
    int fun(int a, int b)
    {
      return (a - b);
    }

4. 编译源码

    [^_^ 11:52 ~/test/gtest/testcase] g++ -c sample.cpp

5. 测试用例代码

  [^_^ 11:52 ~/test/gtest/testcase] vim ut_sample.cpp
  #include "gtest/gtest.h"
  #include "sample.h"
  // TEST (gtest macro),fun:function name to test, "case1" test case name
  TEST(fun, case1)
  {
      EXPECT_LT(-2, fun(1, 2));
      EXPECT_EQ(-1, fun(1, 2));
      ASSERT_LT(-2, fun(1, 2));
      ASSERT_EQ(-1, fun(1, 2));
  }
  int main(int argc, char* argv[])
  {
      testing::InitGoogleTest(&argc, argv);
      return RUN_ALL_TESTS();
  }

6. 编译用例代码

  [^_^ 11:54 ~/test/gtest/testcase] g++ -c ut_sample.cpp -I../include/
  [^_^ 11:54 ~/test/gtest/testcase] g++ ut_sample.o sample.o ../lib/libgtest.a -o sample -lpthread

7. 执行用例

  [^_^ 11:54 ~/test/gtest/testcase] ./sample
  [==========] Running 1 test from 1 test case.
  [----------] Global test environment set-up.
  [----------] 1 test from fun
  [ RUN      ] fun.case1
  [       OK ] fun.case1 (0 ms)
  [----------] 1 test from fun (0 ms total)

  [----------] Global test environment tear-down
  [==========] 1 test from 1 test case ran. (0 ms total)
  [  PASSED  ] 1 test.
