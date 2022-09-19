---
layout: post
title: GDB实战学习笔记
date: 2022-08-24
author: lau
tags: [C++, Blog]
comments: true
toc: false
pinned: false
---
GDB作为经典的调试系统，这里记录它的原理和实践。

<!-- more -->


## GDB概述
### GDB是什么
GDB是GNU开源组织发布的一个强大的UNIX/Linux下的程序调试工具，支持多种语言。
### 编译程序 “-g”选项
使用GDB调试C/C++之前，需要用”-g”选项对程序进行编译，”-g”选项把调试信息加到可执行文件中。
```c++
gcc -g hello.c -o hello
```
### 用GDB启动程序
启动GDB的方式大概有以下3种：
- gdb <program>: Program就是可执行文件，一般放在当前目录中，例如 gdb ./gmserver
- gdb <program> core: 用gdb调试 core dump文件，调试程序非法执行产生的core dump文件
- gdb <program> pid: 如果要调试的程序已经启动了，也可以制定进程的运行的pid，gdb会自动关联起来。例如 gdb ./gmserver 19131
GDB启动以后，就可以输入run (r)启动程序了，注意如果是用第三种方式启动的，gdb启动以后要输入continue(c)，否则程序会重新开始执行。

关于core dump，如果要产生core dump，要在终端输入ulimit –c unlimited，这样才能生成 core文件。

### 设置程序启动参数
有一些程序运行时候需要可以传入启动参数，使用gdb调试的时候，我们也同样可以传入启动参数。

例如，启动一个程序 ./gdbtest gauss gmdb，这个例子中启动gdbtest的时候输入了两个参数gauss和gmdb，如果我们用gdb启动就可以这样输入：set args gauss gmdb。

查看程序启动参数：
```shell
show args
```
### 查看源文件: list
- list <linenum> 行号
- list <+offset> 当前行号的正偏移量
- list <-offset> 当前行号的负偏移量
- list <filename:linenum> 哪个文件的哪一行
- list <function> 函数名
- list <filename:function> 哪个文件中的哪个函数
- <*address> 程序运行时的语句在内存中的地址
- show list 查看当前显示源代码的行数
- set list <count> 设置一次显示源代码的行数
### breakpoint
预先设置的一个让代码在某种条件下、在某个地点停止下来

行号断点
- break gdb_sample:25 / break 25
函数断点
- break gdb_sample:main / break main
条件断点
- break thread_entry_1 if(g_ulCount==10)
内存断点
- break *0x400678，适合于没有debug信息或者没有源文件，比如一些第三方的库。这个时候可以结合反汇编命令 disassemble
### watchpoint
watchpoint是一种特殊的断点，一般用来观察某个表达式\值\地址的内容是否有变化，如果有变化，则停止程序的执行。

设置watchpoint和设置breakpoint的命令不同，但是管理watchpoint的命令和其他breakpoint是一致的。

watchpoint分为硬件watchpoint、软件watchpoint

watch 当表达式（变量）的值被写时，停止程序

rwatch  当表达式（变量）的值被读时，停止程序

awatch 当表达式（变量）的值被读写时，停止程序

### 设置watchpoint
```shell
watch <expr>
watch g_ulCount
```
watch *(address) 监视内存地址，这种设置watchpoint的方法比较有用，因为变量有时候不是被“显式”更改的，可能是直接通过地址修改，或者是由于非法访问（野指针，数组越界）被修改的。或者说，有时候我们要监视的根本就不是某一个变量，可能是代码段的一段地址。
```shell
watch *(&g_ulCount)
```
### 恢复运行
- continue [ignore-count]
恢复程序运行，直到程序结束，或是下一个断点到来。ignore-count表示忽略其后的断点次数
- next <count>
同样单步跟踪，如果有函数调用，他不会进入该函数。后面可以加count也可以不加，不加表示一条条地执行，加表示执行后面的count条指令，然后再停住。
- step <count>
单步跟踪，如果有函数调用，他会进入该函数。进入函数的前提是，此函数被编译有debug信息。后面可以加count也可以不加，不加表示一条条地执行，加表示执行后面的count条指令，然后再停住。
- set step-mode on
打开step-mode模式，于是，在进行单步跟踪时，程序不会因为没有debug信息而不停住。这个参数有很利于查看机器码。
- until 或 u
当你厌倦了在一个循环体内单步跟踪时，这个命令可以运行程序直到退出循环体。
- stepi 或 si
- nexti 或 ni
单步跟踪一条机器指令！一条程序代码有可能由数条机器指令完成，stepi和nexti可以单步执行机器指令。
- finish
运行程序，直到当前函数完成返回。并打印函数返回时的堆栈地址和返回值及参数值等信息。
### 管理断点
- delete n
删除一个断点
- disable n
禁用一个断点
- enable n
启用一个断点

注：n为断点编号

### 查看call stack
当函数被程序调用时，这个函数的信息(函数地址、参数、局部变量)都会被保存到栈中。
- backtrace (bt)
 查看当前函数调用的栈的所有信息
- frame 
栈某一层的信息，可以用Info frame查看详细的信息，程序因为断点而停止时，GDB会自动的输出当前frame的简要信息
- up <n> 表示向栈的上面移动n层
- down <n> 表示向栈的下面移动n层
- info frame
查看当前“栈层”的详细信息
- info args
查看当前函数调用的参数
- info local
查看当前函数中所有局部变量以及其值
### 查看数据
在调试程序时，当程序被停住时，可以使用print命令(简写p)来查看当前程序中的数据。print的命令格式为：
- print /<f> <expr>         print 可以输出变量或者表达式的值
gdb 可以随时查看一下三种变量：
- 全局变量
- 静态全局变量 （本文件可见）
- 局部变量
输出格式
- x  16进制 d  十进制  o 八进制 t 二进制 c 字符格式 f 浮点数格式
- u 十进制无符号 a 输出一个地址保存的符号
### multiple thread
info thread 查看当前程序中线程的情况

thread n  切换thread id为 n 的线程为当前线程

注： 程序停止下来的时候，正在执行的线程前面有 *

这个功能可以用在下面这个场景：
有的时候程序因为某些原因出错了（例如非字节对齐访问，除0），系统就会产生一个错误，而操作系统可能会处理这个错误，然后停止在一个操作系统的线程里面。这个时候我们用bt查看call stack，并不是真正出错的地方，因为我们需要用info thread查看一下所有线程的情况，找出出错的那个线程，然后thread n跳转到出错的线程，再用bt查看call stack。 在嵌入式系统中会常用这样的方法，因为嵌入式可能操作系统也挂掉了。
### multiple thread breakpoints
有一些公共类函数可能会被很多个线程调用，如果我们需要查看该函数被某个线程调用的情况，那么我们需要设置断点仅针对某个ID的线程，例如：
```shell
break printf thread 3
```



