---
layout: post
title: Shell学习实践笔记
date: 2022-08-25
author: lau
tags: [Shell, Blog]
comments: true
toc: false
pinned: false
---
Shell在执行批量任务时编写脚本效率很高，这里记录学习笔记。

<!-- more -->

## Shell的历史
Shell的作用是解释执行用户的命令，用户输入一条命令，Shell就解释执行一条，这种方式称为交互式（Interactive），Shell还有一种执行命令的方式称为批处理（Batch），用户事先写一个Shell脚本（Script），其中有很多条命令，让Shell一次把这些命令执行完，而不必一条一条地敲命令。Shell脚本和编程语言很相似，也有变量和流程控制语句，但Shell脚本是解释执行的，不需要编译，Shell程序从脚本中一行一行读取并执行这些命令，相当于一个用户把脚本中的命令一行一行敲到Shell提示符下执行。

Shell由于历史原因有很多分类，一般常用的是：bash、zsh。

用户在命令行输入命令后，一般情况下Shell会fork并exec该命令，但是Shell的内建命令例外，执行内建命令相当于调用Shell进程中的一个函数，并不创建新的进程。以前学过的cd、alias、umask、exit等命令即是内建命令，凡是用which命令查不到程序文件所在位置的命令都是内建命令，内建命令没有单独的man手册，要在man手册中查看内建命令，应该$ man bash-builtins如export、shift、if、eval、[、for、while等等。内建命令虽然不创建新的进程，但也会有Exit Status，通常也用0表示成功非零表示失败，虽然内建命令不创建新的进程，但执行结束后也会有一个状态码，也可以用特殊变量$?读出。

### Shell简单实例

编写一个简单的脚本test.sh：
```shell
#! /bin/bash
cd ..
ls
```
Shell脚本中用#表示注释，相当于C语言的//注释。但如果#位于第一行开头，并且是#!（称为Shebang）则例外，它表示该脚本使用后面指定的解释器/bin/sh解释执行。如果把这个脚本文件加上可执行权限然后执行：
```shell
chmod a+x test.sh
./test.sh
```

Shell会fork一个子进程并调用exec执行./test.sh这个程序，exec系统调用应该把子进程的代码段替换成./test.sh程序的代码段，并从它的_start开始执行。然而test.sh是个文本文件，根本没有代码段和_start函数，怎么办呢？其实exec还有另外一种机制，如果要执行的是一个文本文件，并且第一行用Shebang指定了解释器，则用解释器程序的代码段替换当前进程，并且从解释器的_start开始执行，而这个文本文件被当作命令行参数传给解释器。因此，执行上述脚本相当于执行程序
```shell
/bin/sh ./test.sh
```

如果将命令行下输入的命令用()括号括起来，那么也会fork出一个子Shell执行小括号中的命令，一行中可以输入由分号;隔开的多个命令，比如：
```shell
(cd ..;ls -l)
```

和上面两种方法执行Shell脚本的效果是相同的，cd ..命令改变的是子Shell的PWD，而不会影响到交互式Shell。然而命令
```shell
cd ..;ls -l
```

则有不同的效果，cd ..命令是直接在交互式Shell下执行的，改变交互式Shell的PWD，然而这种方式相当于这样执行Shell脚本：
```shell
source ./test.sh
```
source或者.命令是Shell的内建命令，这种方式也不会创建子Shell，而是直接在交互式Shell下逐行执行脚本中的命令。

## 基本语法
### 变量
变量全部由大写字母组成，分为环境变量和本地变量。

### 命令代换：`或 $()
由'`'反引号括起来的也是一条命令，Shell先执行该命令，然后将输出结果立刻代换到当前命令行中。命令代换也可以用$()表示。

### 算术代换：$(())
用于算术计算，$(())中的Shell变量取值将转换成整数，同样含义的$[]等价。

### 转义字符\
和C语言类似，\在Shell中被用作转义字符，用于去除紧跟其后的单个字符的特殊意义（回车除外），换句话说，紧跟其后的字符取字面值。

### 单引号
和C语言不一样，Shell脚本中的单引号和双引号一样都是字符串的界定符（双引号下一节介绍），而不是字符的界定符。单引号用于保持引号内所有字符的字面值，即使引号内的\和回车也不例外，但是字符串中不能出现单引号。如果引号没有配对就输入回车，Shell会给出续行提示符，要求用户把引号配上对。

### 双引号
被双引号用括住的内容，将被视为单一字串。它防止通配符扩展，但允许变量扩展。这点与单引号的处理方式不同。

## Shell脚本语法
### 条件测试：test \[
命令test或\[可以测试一个条件是否成立，如果测试结果为真，则该命令的Exit Status为0，如果测试结果为假，则命令的Exit Status为1（注意与C语言的逻辑表示正好相反）。
### if/then/elif/else/fi
在Shell中用if、then、elif、else、fi这几条命令实现分支控制。
```shell
    #! /bin/bash
    echo "Is it morning? Please answer yes or no."
    read YES_OR_NO
    if [ "$YES_OR_NO" = "yes" ]; then
      echo "Good morning!"
    elif [ "$YES_OR_NO" = "no" ]; then
      echo "Good afternoon!"
    else
      echo "Sorry, $YES_OR_NO not recognized. Enter yes or no."
      exit 1
    fi
    exit 0
```
### case/esac
case命令可类比C语言的switch/case语句，esac表示case语句块的结束。C语言的case只能匹配整型或字符型常量表达式，而Shell脚本的case可以匹配字符串和Wildcard，每个匹配分支可以有若干条命令，末尾必须以;;结束，执行时找到第一个匹配的分支并执行相应的命令，然后直接跳到esac之后，不需要像C语言一样用break跳出。

```shell
  #! /bin/bash
    echo "Is it morning? Please answer yes or no."
    read YES_OR_NO
    case "$YES_OR_NO" in
    yes|y|Yes|YES)
      echo "Good Morning!";;
    [nN]*)
      echo "Good Afternoon!";;
    *)
      echo "Sorry, $YES_OR_NO not recognized. Enter yes or no."
      exit 1;;
    esac
    exit 0
```
### for/do/done
 Shell脚本的for循环结构和C语言很不一样，它类似于某些编程语言的foreach循环。
 ```shell
  #! /bin/bash
    for FRUIT in apple banana pear; do
      echo "I like $FRUIT"
    done
 ```
 FRUIT是一个循环变量，第一次循环$FRUIT的取值是apple，第二次取值是banana，第三次取值是pear。

### while/do/done
while的用法和C语言类似。比如一个验证密码的脚本：
```shell
  #! /bin/bash
    echo "Enter password:"
    read TRY
    while [ "$TRY" != "secret" ]; do
      echo "Sorry, try again"
      read TRY
    done
```

Shell还有until循环，类似C语言的do...while循环。本章从略。

### break和continue

break[n]可以指定跳出几层循环，continue跳过本次循环步，没跳出整个循环。

break跳出，continue跳过。
### 位置参数和特殊变量
有很多特殊变量是被Shell自动赋值的，我们已经遇到了$?和$1，现在总结一下：

常用的位置参数和特殊变量：
```
$0 相当于C语言main函数的argv[0]
$1、$2...    些称为位置参数（Positional Parameter），
$#  相当于C语言main函数的argc - 1，注意这里的#后面不表示注释
$@  表示参数列表"$1" "$2" ...，例如可以用在for循环中的in后面。
$*  表示参数列表"$1" "$2" ...，同上
$?  上一条命令的Exit Status
$$  当前进程号
```

### 函数
Shell中也有函数的概念，但是函数定义中没有返回值也没有参数列表。例如：
```shell
 #! /bin/bash
    foo(){ echo "Function foo is called";}
    echo "-=start=-"
    foo
    echo "-=end=-"
```
注意函数体的左花括号'{'和后面的命令之间必须有空格或换行，如果将最后一条命令和右花括号'}'写在同一行，命令末尾必须有;号。

在定义foo()函数时并不执行函数体中的命令，就像定义变量一样，只是给foo这个名字一个定义，到后面调用foo函数的时候（注意Shell中的函数调用不写括号）才执行函数体中的命令。Shell脚本中的函数必须先定义后调用，一般把函数定义都写在脚本的前面，把函数调用和其它命令写在脚本的最后。

Shell函数没有参数列表并不表示不能传参数，事实上，函数就像是迷你脚本，调用函数时可以传任意个参数，在函数内同样是用$0、$1、$2等变量来提取参数，函数中的位置参数相当于函数的局部变量，改变这些变量并不会影响函数外面的$0、$1、$2等变量。函数中可以用return命令返回，如果return后面跟一个数字则表示函数的Exit Status。

下面这个脚本可以一次创建多个目录，各目录名通过命令行参数传入，脚本逐个测试各目录是否存在，如果目录不存在，首先打印信息然后试着创建该目录。

```shell
 #! /bin/bash
    is_directory()
    {
      DIR_NAME=$1
      if [ ! -d $DIR_NAME ]; then
        return 1
      else
        return 0
      fi
    }
    for DIR in "$@"; do
      if is_directory "$DIR"
      then :
      else
        echo "$DIR doesn't exist. Creating it now..."
        mkdir $DIR > /dev/null 2>&1
        if [ $? -ne 0 ]; then
          echo "Cannot create directory $DIR"
          exit 1
        fi
      fi
    done
```
注意is_directory()返回0表示真返回1表示假。

### shell三剑客
shell里面有三大剑客：grep、awk、sed。后面专门写一篇文章来讲解。
