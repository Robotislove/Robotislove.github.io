---
layout: post
title: LeetCode刷题线段树使用总结学习笔记
date: 2022-09-20
author: lau
tags: [Leetcode, Blog]
comments: true
toc: false
pinned: false




---

LeetCode刷题线段树使用总结学习笔记。

<!-- more -->

## 线段树

[线段树](https://so.csdn.net/so/search?q=线段树&spm=1001.2101.3001.7020)兼具了前缀和数组和差分数组的特点，但线段树同时实现了快速的区间查询和区间更新。本文中区间数据值区间和。

如数组｛1，2，3，4，5｝用线段树表示如下图所示：

![](https://img-blog.csdnimg.cn/4946cfb5d42548c898cdd9ac74112fdb.jpeg#pic_center)

## 创建线段树中的节点

①．概述

线段树中每个节点都代表了一个区间范围，并且包含了左右两个子区间。

```c++
struct Node {
   int value = 0;    //区间的值
   int diff_value = 0;  //数据更新增量 
   int left_bound;//左边界
   int righ_bound;//右边界
   Node* left_child = nullptr;//左节点
   Node* right_child = nullptr;//右节点
   Node(int left, int right) :left_bound(left), righ_bound(right) {}
};
```

②．根节点

线段树中的根节点代表了整个区间范围，因此创建一个线段树必须知道当前序列的确定边界大小。

③．子节点

每个节点代表的区间范围从中间一分为二可以拆分为左右两个子区间，线段树中的叶子节点代表了序列中各元素的值。

子节点可以根据需要动态创建，不使用则可以不创建。

```c++
int mid = (node->left_bound + node->righ_bound) / 2;
if (node->left_child == nullptr) 
    node->left_child = new Node(node->left_bound, mid);
if (node->right_child == nullptr)
    node->right_child = new Node(mid + 1, node->righ_bound);
```

## 更新指定区间的元素值

①．概述

从根节点开始依次遍历树中的各节点：若当前节点在指定的区间范围之外则跳过；若当前节点在指定的区间范围之内，则可以直接更新该节点的值；若当前节点和指定的区间有交集，则需要先更新子节点的值，再根据子节点值更新当前节点。

![](https://img-blog.csdnimg.cn/47ef8da6e3724357b8019556f1a0e021.jpeg#pic_center)

②．更新一个节点的值

如要更新一个节点的值仅更新当前节点即可，无需递归更新所有子节点。保存数据变化的增量，子节点在需要的时候再更新其值。

```c++
void updateNodeValue(Node* node, int val)
{
    node->value += (node->righ_bound - node->left_bound + 1) * val;
    node->diff_value += val;//保存元素值变化增量
}
```

③．根据增量更新各子节点的值

根据当前节点保存的数据变化增量，更新子节点的具体值，更新完把增量重置为 $0$ 。

```c++
void push_down(Node* node) {
  if (node == nullptr) return;
    int mid = (node->left_bound + node->righ_bound) / 2;
    if (node->left_child == nullptr)
        node->left_child = new Node(node->left_bound, mid);
    if (node->right_child == nullptr)
        node->right_child = new Node(mid + 1, node->righ_bound);

    if (node->diff_value == 0) return;//数据无变化

    updateNodeValue(node->left_child, node->diff_value);
    updateNodeValue(node->right_child, node->diff_value);
         
    node->diff_value = 0;
}
```

④．根据子节点的值更新当前节点

当子节点数据变化后，要根据其结果更新当前节点的值。

```c++
void push_up(Node* node) {
    if (node->left_child->value)
         node->value += node->left_child->value;
    if (node->right_child->value)
         node->value += node->right_child->value;
}
```

## 查询指定区间的元素值

①．概述

从根节点开始依次遍历树中的各节点：若当前节点在指定的区间范围之外则返回 0；若当前节点在指定的区间范围之内，则可以返回该节点的值；若当前节点和指定的区间有交集，则需要先查询子节点的值，再根据子节点查询结果返回。

②．代码示例

```c++
int query(Node* node, int left, int right) {
    if (node == nullptr) return 0;
    if (node->left_bound > right || node->righ_bound < left) return 0;//当前区间完全不在范围内
    else if (left <= node->left_bound && node->righ_bound <= right)  return node->value;
    else {
         push_down(node);
         int left_ret = query(node->left_child, left, right);
         int right_ret = query(node->right_child, left, right);
         return left_ret + right_ret;
    }
}
```

## 完整实现代码

```c++
class SegmentTree {
public:
    //线段树节点，每个节点表示一个区间
    struct Node {
        int value = 0;    //节点代表的值
        int diff_value = 0;  //数据更新增量 
        int left_bound;//左边界
        int righ_bound;//右边界
        Node* left_child = nullptr;//左节点
        Node* right_child = nullptr;//右节点
        Node(int left, int right) :left_bound(left), righ_bound(right) {}
    };
private:
    Node* root = nullptr;//根节点，表示完整的区间
public:
    SegmentTree(int left, int right):root(new Node(left, right)) {}
    void update(int left, int right,int val) // [left,right] 内的元素都加 val
    {
        update(root, left, right,  val);
    }
    int query(int left, int right) // 查询 [left,right] 区间和
    {
        return query(root, left, right);
    }
private:
    //更新节点的值
    void updateNodeValue(Node* node, int val)
    {
        node->value += (node->righ_bound - node->left_bound + 1) * val;
        node->diff_value += val;//保存元素值变化增量
    }
    void update(Node* node, int left, int right, int val) {
        if (node == nullptr)   return;

        if (node->left_bound > right || node->righ_bound < left) return;//当前区间完全不在范围内
        else if (left <= node->left_bound && node->righ_bound <=right ) //当前节点，完全包含于需要更新的区间
        {
            updateNodeValue(node, val);  //更新当前节点的值
            return;
        }
        else //有交集
        {
            push_down(node);
            update(node->left_child, left, right,val);
            update(node->right_child, left, right, val);
            push_up(node);          
        }
    }
    void push_down(Node* node) {
        if (node == nullptr) return;

        int mid = (node->left_bound + node->righ_bound) / 2;
        if (node->left_child == nullptr) node->left_child = new Node(node->left_bound, mid);
        if (node->right_child == nullptr)  node->right_child = new Node(mid + 1, node->righ_bound);

        if (node->diff_value == 0) return;//数据无变化

        updateNodeValue(node->left_child, node->diff_value);
        updateNodeValue(node->right_child, node->diff_value);
         
        node->diff_value = 0;
    }

    void push_up(Node* node) {
        if (node->left_child->value)
            node->value += node->left_child->value;
        if (node->right_child->value)
            node->value += node->right_child->value;
    }

    int query(Node* node, int left, int right) {
        if (node == nullptr) return 0;
        if (node->left_bound > right || node->righ_bound < left) return 0;//当前区间完全不在范围内
        else if (left <= node->left_bound && node->righ_bound <= right)  return node->value;
        else {
            push_down(node);
            int left_ret = query(node->left_child, left, right);
            int right_ret = query(node->right_child, left, right);
            return left_ret + right_ret;
        }
    }
   
};
```

