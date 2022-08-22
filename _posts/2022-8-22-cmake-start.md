---
layout: post
title: Cmake使用总结笔记
date: 2022-08-22
author: lau
tags: [Cmake, Blog]
comments: true
toc: true
pinned: false
---

Cmake常见用法。

<!-- more -->

## 概述

完整的Cmake工程一般如下图。

![](http://assets.processon.com/chart_image/6302f50f0e3e7437cac4c9bb.png)

基础命令：

- cmake_minimum_required：cmake最小版本号
- project：工程名、版本
- 单行注释：#
- 多行注释：#[[]]
- 打印：message
- 定义变量：set、option
- 指定子构建目录：add_subdirectory，子目录必须包含CMakeLists.txt
- 变量引用：${VARIABLE_NAME}
- 条件分支：if()-else if()-else()-endif()
- 其他：PROJECT_NAME、PROJECT_SOURCE_DIR、循环分支…

总结：
-  变量使用${}方式取值，但是在IF控制语句中是直接使用变量名
-  指令(参数1 参数2...)，其中指令不区分大小写

## 基础操作
一共八个步骤：
- 设置编译选项: `set(CMAKE_CXX_FLAGS "-std=c++11 $CMAKE_CXX_FLAGS")`
- 指定构建配置: `option` & `configure_file`，定义符号区分代码逻辑
- 指定头文件: `include_directories(./)`
- 生成静态库: 先导入源文件`aux_source_directory`，然后`add_libraries`
- 生成动态库: 同上
- 指定外部lib库: `link_directories()`, `target_link_libraries(${PROJECT_NAME} #文件)`
- 生成可执行目标文件: `add_executable(${PROJECT_NAME} main.cpp)`
- 指定文件存放目录: `set(# 设置变量)` 或者install指令
### 

