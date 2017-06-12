---
layout: post
title: "Compilers"
date: 2017-06-11 21:52:00 
description: "Some basics of compilers"
tag: Compilers
---

### 1. compile tools

### 2. 函数调用过程

#### 2.1 汇编函数调用过程

<center>
<img src="/images/posts/2017-06-11-Compilers/x86-registers.png" width="500" height="400" />
Figure 1. x86寄存器的结构
</center>

调用规则分为两个方面，及调用者规则和被调用者规则，如一个函数A调用一个函数B，则A被称为调用者(Caller)，B被称为被调用者(Callee)。

<center>
<img src="/images/posts/2017-06-11-Compilers/stack-convention.png" width="500" height="400" />
Figure 2. 一个调用过程中的内存中的栈布局
</center>

##### Caller Rules

调用者规则包括一系列操作，描述如下：

1）在调用子程序之前，调用者应该保存调用者的寄存器(eax，ecx，edx)的值。由于被调用的子程序会修改这些寄存器，所以为了在调用子程序完成之后能正确执行，调用者必须在调用子程序之前将这些寄存器的值入栈。

2）在调用子程序之前，将参数入栈。参数入栈的顺序应该是从最后一个参数开始，如上图中parameter3先入栈。

3）利用call指令调用子程序。这条指令将返回地址放置在参数的上面，并进入子程序的指令执行。（子程序的执行将按照被调用者的规则执行）

当子程序返回时，调用者期望找到子程序保存在eax中的返回地址。为了恢复调用子程序执行之前的状态，调用者应该执行以下操作：

　　3.1）清除栈中的参数；

　　3.1）将栈中保存的eax值、ecx值以及edx值出栈，恢复eax、ecx、edx的值（当然，如果其它寄存器在调用之前需要保存，也需要完成类似入栈和出栈操作）

Example:

	push [var] ; Push last parameter first
	push 216   ; Push the second parameter
	push eax   ; Push first parameter last
	
	call _myFunc ; Call the function (assume C naming)
	
	add esp, 12

如上代码展示了一个调用子程序的调用者应该执行的操作:

- 此汇编程序调用一个具有三个参数的函数_myFunc，其中第一个参数为eax，第二个参数为常数216，第三个参数为var指示的内存中的值。
- 在调用返回时，调用者必须清除栈中的相应内容，在上例中，参数占有12个字节，为了消除这些参数，只需将**ESP**加12即可。
- _myFunc的值保存在**eax**中，ecx和edx中的值也许已经被改变，调用者还必须在调用之前保存在栈中，并在调用结束之后，出栈恢复ecx和edx的值。

##### Callee Rules
1）将ebp入栈，并将esp中的值拷贝到ebp中，其汇编代码如下：

    push ebp
    mov  ebp, esp

上述代码的目的是保存调用子程序之前的基址指针，基址指针用于寻找栈上的参数和局部变量。当一个子程序开始执行时，基址指针保存栈指针指示子程序的执行。为了在子程序完成之后调用者能正确定位调用者的参数和局部变量，ebp的值需要返回。

2）在栈上为局部变量分配空间。

3）保存callee-saved寄存器的值，callee-saved寄存器包括ebx,edi和esi，将ebx,edi和esi压栈。

4）在上述三个步骤完成之后，子程序开始执行，当子程序返回时，必须完成如下工作：

　　4.1）将返回的执行结果保存在eax中

　　4.2）弹出栈中保存的callee-saved寄存器值，恢复callee-saved寄存器的值（ESI和EDI）

　　4.3）收回局部变量的内存空间。实际处理时，通过改变EBP的值即可：mov esp, ebp。 

　　4.4）通过弹出栈中保存的ebp值恢复调用者的基址寄存器值。

　　4.5）执行ret指令返回到调用者程序。

After these three actions are performed, the body of the subroutine may proceed. When the subroutine is returns, it must follow these steps:

Leave the return value in EAX.

Example:

	.486
	.MODEL FLAT
	.CODE
	PUBLIC _myFunc
	_myFunc PROC
	  ; Subroutine Prologue
	  push ebp     ; Save the old base pointer value.
	  mov ebp, esp ; Set the new base pointer value.
	  sub esp, 4   ; Make room for one 4-byte local variable.
	  push edi     ; Save the values of registers that the function
	  push esi     ; will modify. This function uses EDI and ESI.
	  ; (no need to save EBX, EBP, or ESP)
	
	  ; Subroutine Body
	  mov eax, [ebp+8]   ; Move value of parameter 1 into EAX
	  mov esi, [ebp+12]  ; Move value of parameter 2 into ESI
	  mov edi, [ebp+16]  ; Move value of parameter 3 into EDI
	
	  mov [ebp-4], edi   ; Move EDI into the local variable
	  add [ebp-4], esi   ; Add ESI into the local variable
	  add eax, [ebp-4]   ; Add the contents of the local variable
	                     ; into EAX (final result)
	
	  ; Subroutine Epilogue 
	  pop esi      ; Recover register values
	  pop  edi
	  mov esp, ebp ; Deallocate local variables
	  pop ebp ; Restore the caller's base pointer value
	  ret
	_myFunc ENDP
	END


#### 2.2 C函数调用过程

refer to [4,5].


### References:

[1] [X86汇编快速入门](http://www.cnblogs.com/YukiJohnson/archive/2012/10/27/2741836.html), cnblogs.com.

[2] [Memory Dump In Cloud](http://oliveryang.net/), github.com.

[3] [Oliver Yang的博客](http://blog.csdn.net/yayong), csdn.com.

[4] [ｃ语言函数调用过程](http://blog.csdn.net/crazyuo/article/details/40518101), csdn.com.

[5] [深入理解C语言的函数调用过程](http://blog.csdn.net/lee244868149/article/details/49497679), csdn.com.