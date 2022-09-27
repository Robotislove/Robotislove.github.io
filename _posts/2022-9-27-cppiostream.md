---
layout: post
title: C++读取文件总结学习笔记
date: 2022-09-27
author: lau
tags: [C++, Blog]
comments: true
toc: false
pinned: false
---

C++读取文件总结学习笔记。

<!-- more -->

# fstream类。

fstream是菱形继承类，所以他包括的ifstream类和ofstream类。

- ifstream定义读文件的对象。

- ofstream定义写文件的对象。

## open：文件的打开方式。

open打开文件这个函数有两个参数：第一个是我们打开文件的名，第二个是打开的方式。

1）打开文件的方式有：

- app:以可写的方式打开文件并且是在文件内容的末尾进行写入，不会覆盖掉之前文件的内容。
- binary:是以二进制的方式打开文件。
- in:是以只读的方式打开文件不能改变文件的内容。
- out:打开文件进行写的操作，即是把数据写入文件中。
- ate：打开文件是文件指针指向文件的末尾，但是可以在任意的位置进行插入。
- trunc:如果打开的文件已经存在则清空文件的内容。
  

## 应用实例

```c++
#include <iostream>
#include <fstream>
#include <sstream>
using namespace std;

//对文件进行写入的操作
void write()
{
    ofstream file;
    file.open("./1.txt", ios_base::app);

    if (!file.is_open())
    {
        cout << "打开文件失败";
    }
    string ss;
    cin >> ss;
    file << ss << endl;

    file.close();
}

//第一种读取文件的操作
void read1()
{
    ifstream file;

    file.open("./1.txt", ios_base::in);

    if (!file.is_open())
    {
        cout << "打开文件失败";
    }

    string s;
    while (getline(file, s))
    {
        cout << s << endl;
    }
    file.close();
}
```

```c++
//第二种读取文件的操作
void read2()
{
    ifstream file;

    file.open("./1.txt", ios_base::in);

    if (!file.is_open())
    {
        cout << "打开失败";
    }

    stringstream s;
    s << file.rdbuf();

    string str = s.str();
    cout << str << endl;
}

int main()
{
    write();
    read1();
    cout << endl;
    read2();
    cout << endl;
    return 0;
}
```

