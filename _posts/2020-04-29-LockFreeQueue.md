---
layout: post
title: "LockFreeQueue"
date: 2020-04-29 18:15:06 
description: "LockFreeQueue"
tag: Linux
---

## 1.原子操作
- CAS
- Fetch and Add, https://en.wikipedia.org/wiki/Fetch-and-add
- Test and Set, https://en.wikipedia.org/wiki/Test-and-set
- Test and test-and-set, https://en.wikipedia.org/wiki/Test_and_test-and-set

### 1.1 CAS
Compare & Set，或是 Compare & Swap。现在几乎所有的CPU指令都支持CAS的原子操作，X86下对应的是 CMPXCHG 汇编指令。这个操作用C语言来描述就是下面这个样子：（代码来自Wikipedia的Compare And Swap词条）意思就是说，看一看内存*reg里的值是不是oldval，如果是的话，则对其赋值newval。
```c
int compare_and_swap (int* reg, int oldval, int newval)
{
  int old_reg_val = *reg;
  if (old_reg_val == oldval) {
     *reg = newval;
  }
  return old_reg_val;
}
```


https://coolshell.cn/articles/8239.html

https://www.codeproject.com/Articles/153898/Yet-another-implementation-of-a-lock-free-circul
