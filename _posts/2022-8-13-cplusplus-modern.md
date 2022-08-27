---
layout: post
title: 现代C++学习资料汇总笔记
date: 2022-08-13
author: lau
tags: [C++, Archive]
comments: true
toc: true
pinned: false
---

之所以叫现代C++，是因为我发现大部分人学习C++的姿势不太对，个人浅见，用更加现代和国际的观点来对待。

<!-- more -->

## 入门
C++技能树汇总，参考罗能大神：
![](http://assets.processon.com/chart_image/6308b44a07912906e3a5edc5.png)
### 侯捷C++系列教程

- [C++ 面向对象高级开发](https://www.youtube.com/watch?v=GIvZmTfvkJw&list=PLRTJhCIMo8HOIpVXaaI_yqihTuSTPUcCy)
- [C++11/14 新语法新机制](https://www.youtube.com/watch?v=4HZpY7X9RG4&list=PLRTJhCIMo8HM16EbfTa4IgvwgLbj5z_9M)
- [C++ 标准库](https://www.youtube.com/watch?v=IoXNDKyI1L0&list=PLRTJhCIMo8HPW3OA74uVWIFWDCGRVckUV)
- [C++内存大局观](https://www.bilibili.com/video/BV1hY4y1j7DY?spm_id_from=333.337.search-card.all.click)
- 侯捷设计模式大局观，ICT软件大会有，外部无法查看，所以可以看看GeekBand的视频。

### CppCon 2014: Herb Sutter “Back to the Basics! Essentials of Modern C++ Style”

CppCon 的 Back to the Basics 基础系列话题之一，如何编写现代风格的 C++ 代码。
https://www.youtube.com/watch?v=xnqTKD8uD64

### CppCon 2015: Herb Sutter “Writing Good C++14… By Default”

与上一个视频类似，这个着重 C++14 的内容。
https://www.youtube.com/watch?v=hEx5DNLWGgA

### back to Basics: Move Semantics - Klaus Iglberger - CppCon 2019

C++11 引入的移动语义和所有权机制，必看典中典（不懂移动语义可以说就是不懂 C++）。视频有两个部分。
- https://www.youtube.com/watch?v=St0MNEU5b0o
- https://www.youtube.com/watch?v=pIzaZbKUw2s

## 中阶
### 后台高性能并发项目

- [C++高级教程]从零开始开发服务器框架(sylar)

设计和实现一个基于协程的异步网络库，还有后端开发的一些基本组件。看懂这视频需要一定的 Linux 系统编程和网络编程的基础知识，UP 主讲解的并不是很好（几乎没讲解），如果对系统编程和网络编程完全没概念的话是没法看的。
https://www.bilibili.com/video/BV184411s7qF

- co_context 是最近开发的 C++ 异步协程框架

co_context 是最近开发的 C++ 异步协程框架，以易用性为最高目标，尽量兼顾性能。希望从此 C++ 的异步能比 Node.js 更简单，更优雅。

https://codesire-deng.github.io/2022/05/26/co-context-0/

网络编程入门推荐《TCP/IP网络编程》（尹圣雨 著，金国哲 译）这本书，书里分别讲了 Windows 和 Linux 的 Api 用法，只看 Linux 部分就够了。

### Design Patterns: Facts and Misconceptions - Klaus Iglberger - CppCon 2021

思考设计模式在 C++ 的正确实践，Java 派的设计模式并不适合一比一复刻到 C++ 里。

https://www.youtube.com/watch?v=OvO2NR7pXjg

### Breaking Dependencies: The SOLID Principles - Klaus Iglberger - CppCon 2020

设计低耦合高内聚代码架构的基本原则。

https://www.youtube.com/watch?v=Ntraj80qN2k

### Making the Most Out of Your Compiler - Danila Kutenin - CppCon 2021

了解编译器会做哪些优化（其实并不是很全面，仅仅是了解）

https://www.youtube.com/watch?v=tckHl8M3VXM

### Branchless Programming in C++ - Fedor Pikus - CppCon 2021

针对 CPU 分支预测失败率过高情况下的性能优化手法。分支预测是现代 CPU 的一个性能优化机制，在预测成功的情况下确实能大幅提升算力，但是面对一些不确定性过高的场景，预测失败后的回溯成本是非常恐怖的。

https://www.youtube.com/watch?v=g-WPhYREFjk

### MIT 6.172 Performance Engineering of Software Systems, Fall 2018

麻省理工学院的高性能计算入门公开课。可以学到如何根据硬件平台的特点编写最高性能的代码，充分榨取硬件的能力。涉及到基本的编码风格、CPU高速缓存的原理、多线程程序设计和并行计算算法入门等东西。

https://www.youtube.com/watch?v=o7h_sYMk_oc&list=PLUl4u3cNGP63VIBQVWguXxZZi0566y7Wf

## 视频账号

### CppCon

由 C++ 标准委员会与各大 C++ 重度使用企业发起的社区讨论组织。这个视频账号会发布他们的演讲视频。演讲覆盖范围非常广，从最基本的语言功能到深入到各个领域的实际案例都有涵盖。

https://www.youtube.com/user/CppCon

### Meetingcpp
C++ 相关大会

https://meetingcpp.com/2021/

### CppNow

著名 IDE 厂商 JetBrains 发起的 C++ 社区讨论组织。这个视频账号会发布他们的演讲视频。

https://www.youtube.com/user/BoostCon
### C++ Weekly With Jason Turner

每周分享 C++ 碎片知识。经常发布些奇奇怪怪的东西，看个乐呵。

https://www.youtube.com/c/lefticus1

### lazyparser

一个做开源编译器工作的组织（还是企业？），分享非常多编译器相关的东西。2022年7月开始更新《徒手写一个RISC-V编译器》系列视频，讲的挺好的，把观众当成完全不懂编译原理的小 baby 来教。

https://space.bilibili.com/296494084
## 作者
[1] Jeff Preshing，加拿大程序员，主要涉及一些并发编程、语言特性相关话题

[2] Bartosz Milewski，波兰程序员，理论物理学家，参与D语言设计与实现，主要涉及并发编程、模板元编程、范畴论相关话题

[3] Andrzej，主要涉及C++编程实践方面的知识，以及一些关于C++设计和当前发展的理论背景，尤其是概念约束方面

[4] Herb Sutter，C++标准委员会成员，介绍C++的现代化进程、最佳实践

[5] Eric Niebler，C++爱好者，ranges标准库作者，主要涉及C++技巧、函数式编程、协程、结构化并发等话题

[6] Lewis Baker，主要介绍无栈协程理论、特性方面

[7] Bjarne Stroustrup，C++之父，很多论文值得已读，即便他对C++有很强烈想法，也未必能纳入标准，个人觉得他的代码风格很奇怪

[8] 吴咏炜，C++实现Y combinator的代码

[9] 袁英杰，对C++以及软件设计理解、洞察力非常深，尤其是关于C++技术层面的理解

[10] 还有很多优秀的程序员，不再一一枚举，待续

## C++ 20标准
### 类型与对象

https://devblogs.microsoft.com/cppblog/how-to-use-class-template-argument-deduction/

https://herbsutter.com/elements-of-modern-c-style/

https://modern-cpp.readthedocs.io/zh_CN/latest/index.html

https://github.com/elbeno/using-types-effectively/blob/master/presentation.org

### 编译时多态

https://preshing.com/20210315/how-cpp-resolves-a-function-call/

https://accu.org/journals/overload/9/43/frogley_442/

https://www.internalpointers.com/post/quick-primer-type-traits-modern-cpp

http://www.eniscuola.net/en/2016/06/27/the-numbers-of-nature-the-fibonacci-sequence/

https://clang.llvm.org/docs/LanguageExtensions.html#type-trait-primitives

《从数学到泛型编程》

https://eli.thegreenplace.net/2014/sfinae-and-enable_if/

https://www.jianshu.com/p/38f17600f19a

https://zh.wikipedia.org/wiki/访问者模式

https://en.wikipedia.org/wiki/Barton–Nackman_trick

https://ledas.com/post/857-how-to-hack-c-with-templates-and-fri

https://gieseanw.wordpress.com/2019/10/20/we-dont-need-no-stinking-expression-templates/

### 概念约束

C++ Concepts - complete overview

The C++0x Concept Effort

Fundamentals of Generic Programming

The Design and Evolution of C++

Concept checking

Concepts – Design choices for template argument checking

Concepts for C++0x

A concept design (Rev. 1)

Concepts Lite: Constraining Templates with Predicates

Concepts: The Future of Generic Programming

https://akrzemi1.wordpress.com/2020/01/29/requires-expression/

https://akrzemi1.wordpress.com/2020/03/26/requires-clause/

https://akrzemi1.wordpress.com/2020/05/07/ordering-by-constraints/

https://stackoverflow.com/questions/62644070/differences-between-stdis-convertible-and-stdconvertible-to-in-practice

### 元编程介绍

#### 模板元编程

https://ieeexplore.ieee.org/iel5/32/35910/01702623.pdf

#### constexpr元编程

https://www.cppstories.com/2021/constexpr-vecstr-cpp20/

https://www.cppstories.com/2021/constexpr-new-cpp20/#limitations

Don’t constexpr All The Things

https://gist.github.com/Som1Lse/5309b114accc086d24b842fd803ba9d2

https://zh.wikipedia.org/wiki/考拉兹猜想

### ranges标准库

https://ericniebler.com/2018/12/05/standard-ranges/
https://www.nextptr.com/tutorial/ta1208652092/how-cplusplus-rangebased-for-loop-works
https://quuxplusone.github.io/blog/2020/07/11/the-std-swap-two-step/
http://mmore500.com/cse-491/blog/2020/04/20/ranges-transpose.html
https://zh.wikipedia.org/wiki/矩陣乘法
http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2387r0.html
https://github.com/ericniebler/range-v3

### 无栈协程

#### C++ HOPL

The history, present and future of computer language coroutines

https://en.wikipedia.org/wiki/Coroutine#C++

https://www.itproportal.com/features/the-rise-of-the-coroutines/

https://www.youtube.com/watch?v=\_fu0gx-xseY

https://blog.panicsoftware.com/coroutines-introduction/

https://github.com/lewissbaker/lewissbaker.github.io/blob/master/\_posts/2017-09-25-coroutine-theory.md

https://hacksoflife.blogspot.com/2021/06/c-coroutines-getting-past-names.html

Coroutine changes for C++20 and beyond

### 模块

C++ HOPL

https://docs.microsoft.com/en-us/cpp/cpp/modules-cpp?view=msvc-170

Practical C++ Modules

https://vector-of-bool.github.io/2019/03/10/modules-1.html

C++20 modules with GCC11

https://docs.microsoft.com/en-us/cpp/cpp/tutorial-named-modules-cpp?view=msvc-170#create-the-primary-module-interface-unit

### 综合运用

https://docs.python.org/3/library/asyncio.html

## 参考文献
[1] [一个汇总CPP各种资源的网站](https://compile-time.re/)

[2] https://netcan.github.io/archives/
