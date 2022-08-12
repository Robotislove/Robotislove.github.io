---
layout: post
title: Cmake组件化构建大型软件项目松耦合方案笔记
date: 2022-08-12
author: lau
tags: [Cmake, Note]
comments: true
toc: true
pinned: false
---

Cmake组件化构建大型软件项目松耦合方案。

<!-- more -->

## CMake的背景知识

这里简短介绍一下makefile的原理。

生成hello（hello.exe）所需要执行的bash命令：`gcc -v -o hello hello.c `。

该过程可拆解为4个步骤：预处理、编译、汇编、链接。大部分情况下，不需要拆分成这四步来完成文件的生成，一般都是直接生成二进制.o文件，然后链接成可执行文件（或者动态库.so .dll 或者静态库.a .lib）。

![](https://pic1.zhimg.com/v2-8c46167e41d40f14e6f2ea02b4edacfe_1440w.jpg?source=172ae18b)

具体的Makefile编写规则可以参考：[九张图记住Makefile](https://zhuanlan.zhihu.com/p/163287897)。

传统Linux的构建工具是Makefile，功能极其强大但是却存在一些缺点。

- 难以书写，维护者需要关注太多的细节
- 不能跨平台

CMake自身并不能完成构建，它只会生成构建所需的Makefile，最终的构建仍然由Makefile来完成。

CMake的优势
- 屏蔽了Makefile中的很多细节，使得编写更为简单，如.h和.c的依赖
- 跨平台：Linux、Windows、Macos、Unix
- Out-of source build：无需任何特别设置，就能将所有的中间文件存放在一个临时目录，clean简直太轻松了—— 这一点可比Linux Kernel玩得漂亮，是强迫症患者的福音！

当然还有其他和CMake类似的工具，如autotools...

CMake使用的情况：Android App中的so，各种知名开源库：KDE 4、OpenCV、dlib、Qt...

## 一个简单的CMake


其实这不是最简单的CMakeLists.txt，CMake中变量的赋值和引用的表达方式是不一样的，一个Project中可以包含多个executables、libraries。CMake会生成一大堆你不想看的中间文件，我一般是单独建立一个和根CMakeLists.txt同级的子目录build，然后到里面去键入“cmake ../”
。`aux_source_directory(. FULL_SRCS)`可以将当前目录下的所有c文件添加到变量FULL_SRCS中。如果仅仅修改过源代码，却没有修改过CMakeLists.txt，是不需要重新CMake的，直接make就是了
可以让这两个c文件include一个.h文件，然后修改.h文件的内容，直接make，看看自动依赖是否生效。








