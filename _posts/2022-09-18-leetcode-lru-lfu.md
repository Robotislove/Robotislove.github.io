---
layout: post
title: LeetCode刷题LFU缓存、STL容器使用总结学习笔记
date: 2022-09-18
author: lau
tags: [Leetcode, Blog]
comments: true
toc: false
pinned: false


---

LeetCode刷题LFU缓存、STL容器使用总结学习笔记。

<!-- more -->

## 一．背景
容器的概念

一个容器就是一些特定类型对象的集合；它可以保存其他对象或者指向其他对象的指针，还提供了一系列处理其他对象的方法；

容器类自动申请和释放内存，因此无需new和delete操作；

容器本质的作用是对数据进行存储，学习容器的关键是掌握各种容器的特点及其对数据的增删改查的操作方法。

容器的种类

①．顺序容器

 顺序容器为成员提供了控制元素存储以及的快速顺序访问元素的能力；元素顺序和元素值无关，依赖于元素添加时的次序。

- vector：可变大小数组，从末尾快速的插入和删除，支持直接访问任何元素。

- list：双向链表，从任何位置快速的插入和删除，不支持随机访问。
  ![](https://img-blog.csdnimg.cn/20200527163934322.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpY2hhbzQxMA==,size_16,color_FFFFFF,t_70#pic_center)

②．容器适配器

 一个适配器是一种机制，能使某种事物的行为看起来像另外一种事物一样。适配器为顺序容器提供了不同的接口，可以看作是：它保存一个容器，用这个容器再保存所有元素。

- stack：先进后出。

- queue：先进先出。

- priority_queue：最高优先级的元素第一个出列
  

![](https://img-blog.csdnimg.cn/20200527164013222.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpY2hhbzQxMA==,size_16,color_FFFFFF,t_70#pic_center)

③．关联容器

 关联容器支持高效的关键字查找和访问，各元素之间没有严格的物理上的顺序关系。

- **map**：基于关键字查找，不允许重复，所有元素是有序的。

- **set**：快速查找，不允许重复，所有元素是有序的。

![](https://img-blog.csdnimg.cn/20200527164101471.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpY2hhbzQxMA==,size_16,color_FFFFFF,t_70#pic_center)

④．无序容器

 无序容器是内部采用哈希表实现的map或者set。

- **unordered_map**：基于关键字查找，不允许重复，所有元素是有序的。

- **unordered_set**：快速查找，不允许重复，所有元素是有序的。

## 二．顺序容器的使用

1. 引用头文件

   每个容器都定义在一个头文件中，文件名与类型名相同。

| 容器类型     | 头文件                  |
| ------------ | ----------------------- |
| vector       | #include                |
| deque        | #include                |
| list         | #include                |
| forward_list | #include <forward_list> |
| array        | #include                |
| string       | #include                |

2. 迭代器的使用

①．概述

 迭代器是一种检查容器内元素并遍历元素的数据类型，类似指针的操作。每一种标准容器都定义了一种迭代器类型，容器迭代器的范围左闭右开，[begin,end)

②．基本操作

![](https://img-blog.csdnimg.cn/20200527164240803.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpY2hhbzQxMA==,size_16,color_FFFFFF,t_70#pic_center)

3. 顺序容器特点总结

![](https://img-blog.csdnimg.cn/20200604150907259.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpY2hhbzQxMA==,size_16,color_FFFFFF,t_70#pic_center)

4. 关键接口

![](https://img-blog.csdnimg.cn/20200527164316779.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpY2hhbzQxMA==,size_16,color_FFFFFF,t_70#pic_center)

5. string的其他操作

①．基本概念

![](https://img-blog.csdnimg.cn/20200527164339844.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpY2hhbzQxMA==,size_16,color_FFFFFF,t_70#pic_center)

②．string的操作

![](https://img-blog.csdnimg.cn/20200527164353490.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpY2hhbzQxMA==,size_16,color_FFFFFF,t_70#pic_center)

## 三．容器适配器的使用

1. 引用头文件

| 适配器类型     | 头文件   |
| -------------- | -------- |
| stack          | #include |
| queue          | #include |
| priority_queue | #include |

2. 适配器特点总结

![](https://img-blog.csdnimg.cn/20200527164409849.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpY2hhbzQxMA==,size_16,color_FFFFFF,t_70#pic_center)

3. 关键数据接口

![](https://img-blog.csdnimg.cn/20200527164438672.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpY2hhbzQxMA==,size_16,color_FFFFFF,t_70#pic_center)

## 四．有序容器的使用

1. 引用头文件

| 容器类型 | 头文件   |
| -------- | -------- |
| map      | #include |
| set      | #include |
| multimap | #include |
| multiset | #include |

2. pair的使用

①．概述

 pair是将两个数据组合成一个数据。头文件定义在 utility 中。

②．基本操作

![](https://img-blog.csdnimg.cn/20200527164454449.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpY2hhbzQxMA==,size_16,color_FFFFFF,t_70#pic_center)

3. 有序容器特点总结

![](https://img-blog.csdnimg.cn/20200527164529778.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpY2hhbzQxMA==,size_16,color_FFFFFF,t_70#pic_center)

4. 关键数据接口

![](https://img-blog.csdnimg.cn/20200527164556824.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpY2hhbzQxMA==,size_16,color_FFFFFF,t_70#pic_center)

5. 无序容器的使用

每个有序容器都有对应的无序版本，无序容器的操作同其有序版本，但内容元素是无序的，通常效率更高点。

无序容器的头文件定义在 unordered_map 和 unordered_set 中。

## 五．一些常用算法

1. 概述

   算法不直接操作容器，也不依赖于具体的容器，只会运行于迭代器之上。

   大多数算法的头文件定义在 algorithm 中。

2. 常用算法

![](https://img-blog.csdnimg.cn/20200527164612768.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpY2hhbzQxMA==,size_16,color_FFFFFF,t_70#pic_center)

## 五．应用举例

1. LRU缓存机制

```
运用你所掌握的数据结构，设计和实现一个 LRU (最近最少使用) 缓存机制。它应该支持以下操作： 获取数据 get 和 写入数据 put 。

获取数据 get(key) - 如果关键字 (key) 存在于缓存中，则获取关键字的值（总是正数），否则返回 -1。
写入数据 put(key, value) - 如果关键字已经存在，则变更其数据值；如果关键字不存在，则插入该组「关键字/值」。当缓存容量达到上限时，它应该在写入新数据之前删除最久未使用的数据值，从而为新的数据值留出空间。

进阶:

你是否可以在 O(1) 时间复杂度内完成这两种操作？
```

①．题目分析

 O(1)的时间查询数据选择map；O(1)的时间插入\删除数据选择list。

 创建键-值map用于查询数据，保存为key和list中的迭代器对应关系；创建n个list用于保存相同频率的节点，可以快速的插入和删除数据。

②．代码示例

```c++
struct Node
{
    int key;
    int value;
    Node(int m_key,int m_value):key(m_key),value(m_value){}
};
class LRUCache {
public:
    unordered_map<int,list<Node>::iterator> key_map;//k-v键值对
    list<Node> nodelist;
    int m_capatity = 0;
   
    
    LRUCache(int capacity) {
        m_capatity = capacity;
        nodelist.clear();
        key_map.clear();
    }
    
    int get(int key) {
        if( m_capatity == 0) return -1;
        if( key_map.find(key) == key_map.end())
            return -1;
        else
        {
            list<Node>::iterator node = key_map[key] ;
            int val = node->value;
            nodelist.erase(node);//先删除
            nodelist.push_front(Node(key,val));//插入到list开头
            key_map[key] = nodelist.begin();
            return val;
        } 
    }
    
    void put(int key, int value) {
        if( m_capatity == 0) return ;
        if( key_map.find(key) != key_map.end())
        {
            list<Node>::iterator node = key_map[key];
            nodelist.erase(node);
            nodelist.push_front(Node(key,value));
            key_map[key] = nodelist.begin();
        }
        else
        {
            if( key_map.size() == m_capatity)
            {
               
                 key_map.erase(nodelist.back().key );
                 nodelist.pop_back();
            }
            nodelist.push_front(Node(key,value));
            key_map[key] = nodelist.begin();
        }

    }
};

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache* obj = new LRUCache(capacity);
 * int param_1 = obj->get(key);
 * obj->put(key,value);
 */

```

2. LFU缓存

```
请你为最不经常使用（LFU）缓存算法设计并实现数据结构。它应该支持以下操作：get 和 put。

get(key) - 如果键存在于缓存中，则获取键的值（总是正数），否则返回 -1。
put(key, value) - 如果键已存在，则变更其值；如果键不存在，请插入键值对。当缓存达到其容量时，则应该在插入新项之前，使最不经常使用的项无效。在此问题中，当存在平局（即两个或更多个键具有相同使用频率）时，应该去除最久未使用的键。
「项的使用次数」就是自插入该项以来对其调用 get 和 put 函数的次数之和。使用次数会在对应项被移除后置为 0 。

进阶：
你是否可以在 O(1) 时间复杂度内执行两项操作？
```

①．题目分析

 O(1)的时间查询数据选择map；O(1)的时间插入\删除数据选择list。

 创建键-值map用于查询数据，保存为key和list中的迭代器对应关系；创建list保存节点，用于快速的插入和删除数据，当前节点更新时删除节点，然后在list开头插入新节点。

②．代码示例
```c++
struct Node// 双向链表list中的节点 
{
    int key,val,freq;
    Node(int m_key,int m_val,int m_freq):key(m_key),val(m_val),freq(m_freq){}
};
class LFUCache {
    int minfreq,m_cpacity;
    unordered_map<int,list<Node>::iterator> key_table;// key-list中指针
    unordered_map<int,list<Node>> freq_table;//频率-list
public:
    LFUCache(int capacity) {
        minfreq = 0;
        m_cpacity = capacity;
        key_table.clear();
        freq_table.clear();
    }
    
    int get(int key) {
        if( m_cpacity == 0) return -1;
        if( key_table.find(key) == key_table.end())
        {
            return -1;
        }
        list<Node>::iterator node = key_table[key];//找到节点
        int val = node->val,freq = node->freq;
        freq_table[freq].erase(node);//从频率为freq的list中删除node
        if( freq_table[freq].size() == 0)
        {
            freq_table.erase(freq);
            if( minfreq == freq)  minfreq++;
        }
        freq_table[freq+1].push_front(Node(key,val,freq+1));//插入到freq 链表头部
        key_table[key] = freq_table[freq+1].begin();//更新k-v map
        return val;
        
    }
    
    void put(int key, int value) {
        if( m_cpacity == 0) return ;
        if( key_table.find(key) != key_table.end()) //key 已存在
        {
            list<Node>::iterator node = key_table[key];
            int val = node->val,freq = node->freq;
            freq_table[freq].erase(node);
            if( freq_table[freq].size() == 0)
            {
                freq_table.erase(freq);
                if( minfreq == freq) minfreq++;
            }
            freq_table[freq+1].push_front(Node(key,value,freq+1));
            key_table[key] = freq_table[freq+1].begin();
        }
        else //key 不存在
        {
            if( key_table.size() == m_cpacity)//缓存已满
            {
                auto  node = freq_table[minfreq].back();
                key_table.erase(node.key);//删除节点
                freq_table[minfreq].pop_back();//删除尾节点
                if( freq_table[minfreq].size() == 0)
                    freq_table.erase(minfreq);
            }
            freq_table[1].push_front(Node(key,value,1));
            key_table[key] = freq_table[1].begin();
            minfreq = 1;
        }
    }
};

/**
 * Your LFUCache object will be instantiated and called as such:
 * LFUCache* obj = new LFUCache(capacity);
 * int param_1 = obj->get(key);
 * obj->put(key,value);
 */

```

