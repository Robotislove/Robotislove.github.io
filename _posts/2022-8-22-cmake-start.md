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
### 项目实战
main.cpp
```c++
#include <iostream>

#include "out_include_pow.h"
#include "self_static_add.h"
#include "self_static_sub.h"
#include "self_dynamic_div.h"
#include "self_dynamic_mul.h"

int main(int, char**) {
    std::cout << "out: pow(5,2) is " << out_pow(5, 2) << std::endl;
    std::cout << "self_static: add(5, 2) is " << self_static_add(5, 2) << std::endl;
    std::cout << "self_static: sub(5, 2) is " << self_static_sub(5, 2) << std::endl;
    std::cout << "self_dynamic: div(5, 2) is " << self_dynamic_div(5, 2) << std::endl;
    std::cout << "self_dynamic: mul(5, 2) is " << self_dynamic_mul(5, 2) << std::endl;
    return 0;
}
```
主项目CMakeLists:
```cmake
cmake_minimum_required(VERSION 3.0.0)
project(CMAKE_NEW_TEST VERSION 0.1.0)

set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}") #编译选项

include_directories(./out_include 
                    ./self_static/self_static_add/include 
                    ./self_static/self_static_sub/include
                    ./self_dynamic/self_dynamic_div/include
                    ./self_dynamic/self_dynamic_mul/include)

#add_subdirectory(out_src)
##[[
add_subdirectory(self_static)
add_subdirectory(self_dynamic)

link_directories(./out_static ./out_dynamic)

set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin) #在build中执行，相对路径就是相对build。复杂过程使用install
add_executable(${PROJECT_NAME} main.cpp)

option(USE_MY_STATIC_OUT "option" ON) # 如果变量定义过，则不处理；如果未定义过，则设置默认值
option(TEST_CMAKE_DEFINE_OFF_EFFECT "comment" OFF)
set(TEST_VARIABLE_VALUE 0)
configure_file (
  "${PROJECT_SOURCE_DIR}/config/config.h.in"
  "${PROJECT_SOURCE_DIR}/config/config_my89757.h"
  )

message("!!!!!!!!!!!!:" ${USE_MY_STATIC_OUT} "-----------"${CMAKE_INSTALL_PREFIX})
if(${USE_MY_STATIC_OUT} STREQUAL "ON")
target_link_libraries(${PROJECT_NAME} out_static 
                                      self_static_lib 
                                      self_dynamic_dll)
else()
target_link_libraries(${PROJECT_NAME} out_dynamic self_static_lib self_dynamic_dll)
endif()
#]]
```
config.in
```c++
#ifndef CONFIG_H
#define CONFIG_H

#cmakedefine USE_MY_STATIC_OUT
#cmakedefine TEST_CMAKE_DEFINE_OFF_EFFECT
#define TEST_VARIABLE_NAME ${TEST_VARIABLE_VALUE}

#endif
```

动态库CMakeLists:
```cmake
include_directories(${PROJECT_SOURCE_DIR}/out_include)

aux_source_directory(. SRC_FILES)

MESSAGE("-----------------------------------")
SET(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/out_dynamic)

add_library(out_dynamic SHARED ${SRC_FILES})
```

设置全局属性：
```cmake
aux_source_directory(${PROJECT_SOURCE_DIR}/self_dynamic/self_dynamic_div/src 
                    all_src_files)
set_property(GLOBAL PROPERTY SELF_DYNAMIC_DIV_INNER ${all_src_files})
```

```cmake
add_subdirectory(self_dynamic_div)
add_subdirectory(self_dynamic_mul/src)

get_property(SELF_DYNAMIC_DIV GLOBAL PROPERTY "SELF_DYNAMIC_DIV_INNER")
get_property(SELF_DYNAMIC_MUL GLOBAL PROPERTY "SELF_DYNAMIC_MUL_INNER")

include_directories(${PROJECT_SOURCE_DIR}/self_dynamic/comm)

set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
add_library(self_dynamic_dll SHARED ${SELF_DYNAMIC_DIV} ${SELF_DYNAMIC_MUL})
```


