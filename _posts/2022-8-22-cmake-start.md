---
layout: post
title: Cmake使用总结笔记
date: 2022-08-22
author: lau
tags: [Cmake, Blog]
comments: true
toc: false
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

```cmake

1、指定 cmake 的最小版本
cmake_minimum_required(VERSION 3.4.1)

2、设置项目名称，它会引入两个变量 demo_BINARY_DIR 和 demo_SOURCE_DIR，同时，cmake 自动定义了两个等价
的变量 PROJECT_BINARY_DIR 和 PROJECT_SOURCE_DIR。
project(demo)

3、设置编译类型，add_library 默认生成是静态库
add_executable(demo demo.cpp) # 生成可执行文件
add_library(common STATIC util.cpp) # 生成静态库
add_library(common SHARED util.cpp) # 生成动态库或共享库
以上命令将生成：
在 Linux 下是：
        demo
        libcommon.a
        libcommon.so
在 Windows 下是：
        demo.exe
        common.lib
        common.dll

4、明确指定包含哪些源文件
add_library(demo demo.cpp test.cpp util.cpp)

5、设置变量
5.1 set 直接设置变量的值
set(SRC_LIST main.cpp test.cpp)
add_executable(demo ${SRC_LIST})
set(ROOT_DIR ${CMAKE_SOURCE_DIR}) #CMAKE_SOURCE_DIR默认为当前cmakelist.txt目录

5.2 set追加设置变量的值
set(SRC_LIST main.cpp)
set(SRC_LIST ${SRC_LIST} test.cpp)
add_executable(demo ${SRC_LIST})

5.3 list追加或者删除变量的值
set(SRC_LIST main.cpp)
list(APPEND SRC_LIST test.cpp)
list(REMOVE_ITEM SRC_LIST main.cpp)
add_executable(demo ${SRC_LIST})

6、搜索文件
6.1 搜索当前目录下的所有.cpp文件，并命名为SRC_LIST，它会查找目录下的.c,.cpp ,.mm,.cc 等等C/C++语言后缀的文件名
aux_source_directory(. SRC_LIST) 
add_library(demo ${SRC_LIST})

6.2 自定义搜索规则
aux_source_directory(. SRC_LIST)
aux_source_directory(protocol SRC_PROTOCOL_LIST)
add_library(demo ${SRC_LIST} ${SRC_PROTOCOL_LIST})
或者
file(GLOB SRC_LIST "*.cpp" "protocol/*.cpp")
add_library(demo ${SRC_LIST})
# 或者
file(GLOB SRC_LIST "*.cpp")
file(GLOB SRC_PROTOCOL_LIST "protocol/*.cpp")
add_library(demo ${SRC_LIST} ${SRC_PROTOCOL_LIST})
# 或者
file(GLOB_RECURSE SRC_LIST "*.cpp") #递归搜索
FILE(GLOB SRC_PROTOCOL RELATIVE "protocol" "*.cpp") # 相对protocol目录下搜索
add_library(demo ${SRC_LIST} ${SRC_PROTOCOL_LIST})

7、设置包含的目录，头文件目录
include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_BINARY_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/include
)
Linux 下还可以通过如下方式设置包含的目录
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -I${CMAKE_CURRENT_SOURCE_DIR}")

8、设置链接库搜索目录
link_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}/libs
)
Linux 下还可以通过如下方式设置包含的目录
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -L${CMAKE_CURRENT_SOURCE_DIR}/libs")

9、设置 target 需要链接的库
9.1 指定链接动态库或静态库
target_link_libraries(demo libface.a) # 链接libface.a
target_link_libraries(demo libface.so) # 链接libface.so

9.2 指定全路径
target_link_libraries(demo ${CMAKE_CURRENT_SOURCE_DIR}/libs/libface.a)
target_link_libraries(demo ${CMAKE_CURRENT_SOURCE_DIR}/libs/libface.so)

9.3 指定链接多个库
target_link_libraries(demo
    ${CMAKE_CURRENT_SOURCE_DIR}/libs/libface.a
    boost_system.a
    boost_thread
    pthread)

10、打印信息
message(${PROJECT_SOURCE_DIR})
message("build with debug mode")
message(WARNING "this is warnning message")
message(FATAL_ERROR "this build has many error") # FATAL_ERROR 会导致编译失败

11.包含其它 cmake 文件
include(./common.cmake) # 指定包含文件的全路径
include(def) # 在搜索路径中搜索def.cmake文件
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake) # 设置include的搜索路径


12、条件控制
12.1 if…elseif…else…endif
逻辑判断和比较：
if (expression)：expression 不为空（0,N,NO,OFF,FALSE,NOTFOUND）时为真
if (not exp)：与上面相反
if (var1 AND var2)
if (var1 OR var2)
if (COMMAND cmd)：如果 cmd 确实是命令并可调用为真
if (EXISTS dir) if (EXISTS file)：如果目录或文件存在为真
if (file1 IS_NEWER_THAN file2)：当 file1 比 file2 新，或 file1/file2 中有一个不存在时为真，文件名需使用全路径
if (IS_DIRECTORY dir)：当 dir 是目录时为真
if (DEFINED var)：如果变量被定义为真
if (var MATCHES regex)：给定的变量或者字符串能够匹配正则表达式 regex 时为真，此处 var 可以用 var 名，也可以用 ${var}
if (string MATCHES regex)

数字比较：
if (variable LESS number)：LESS 小于
if (string LESS number)
if (variable GREATER number)：GREATER 大于
if (string GREATER number)
if (variable EQUAL number)：EQUAL 等于
if (string EQUAL number)

字母表顺序比较：
if (variable STRLESS string)
if (string STRLESS string)
if (variable STRGREATER string)
if (string STRGREATER string)
if (variable STREQUAL string)
if (string STREQUAL string)

12.2 while…endwhile
12.3 foreach…endforeach
foreach(i RANGE 1 9 2)
    message(${i})
endforeach(i)
# 输出：13579

13、常用变量
13.1 预定义变量
PROJECT_SOURCE_DIR：工程的根目录
PROJECT_BINARY_DIR：运行 cmake 命令的目录，通常是 ${PROJECT_SOURCE_DIR}/build
PROJECT_NAME：返回通过 project 命令定义的项目名称
CMAKE_CURRENT_SOURCE_DIR：当前处理的 CMakeLists.txt 所在的路径
CMAKE_CURRENT_BINARY_DIR：target 编译目录
CMAKE_CURRENT_LIST_DIR：CMakeLists.txt 的完整路径
CMAKE_CURRENT_LIST_LINE：当前所在的行
CMAKE_MODULE_PATH：定义自己的 cmake 模块所在的路径，SET(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)，然后可以用INCLUDE命令来调用自己的模块
EXECUTABLE_OUTPUT_PATH：重新定义目标二进制可执行文件的存放位置
LIBRARY_OUTPUT_PATH：重新定义目标链接库文件的存放位置

13.2 环境变量
$ENV{Name}
set(ENV{Name} value) # 这里没有“$”符号

13.3 系统信息
CMAKE_MAJOR_VERSION：cmake 主版本号，比如 3.4.1 中的 3
­CMAKE_MINOR_VERSION：cmake 次版本号，比如 3.4.1 中的 4
­CMAKE_PATCH_VERSION：cmake 补丁等级，比如 3.4.1 中的 1
­CMAKE_SYSTEM：系统名称，比如 Linux-­2.6.22
­CMAKE_SYSTEM_NAME：不包含版本的系统名，比如 Linux
­CMAKE_SYSTEM_VERSION：系统版本，比如 2.6.22
­CMAKE_SYSTEM_PROCESSOR：处理器名称，比如 i686
­UNIX：在所有的类 UNIX 平台下该值为 TRUE，包括 OS X 和 cygwin
­WIN32：在所有的 win32 平台下该值为 TRUE，包括 cygwin

14、主要开关选项
BUILD_SHARED_LIBS：这个开关用来控制默认的库编译方式，如果不进行设置，使用 add_library 又没有指定库类型的情况下，默认编译生成的库都是静态库。如果 set(BUILD_SHARED_LIBS ON) 后，默认生成的为动态库
CMAKE_C_FLAGS：设置 C 编译选项，也可以通过指令 add_definitions() 添加
CMAKE_CXX_FLAGS：设置 C++ 编译选项，也可以通过指令 add_definitions() 添加
add_definitions(-DENABLE_DEBUG -DABC) # 参数之间用空格分隔
```

