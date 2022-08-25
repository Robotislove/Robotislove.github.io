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

## 参考文献
[1] [一个汇总CPP各种资源的网站](https://compile-time.re/)
