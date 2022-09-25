---
layout: post
title: LeetCode刷题并查集学习笔记
date: 2022-09-19
author: lau
tags: [Leetcode, Blog]
comments: true
toc: false
pinned: false

---

LeetCode刷题并查集学习笔记。

<!-- more -->

## 一．[并查集](https://so.csdn.net/so/search?q=并查集&spm=1001.2101.3001.7020)

顾名思义：并：就是合并，查：就是查找，集：就是集合。并查集是一种树形的[数据结构](https://so.csdn.net/so/search?q=数据结构&spm=1001.2101.3001.7020)，支持以下两种操作：

- 查找：确定某个元素处于哪个子集
- 合并：将两个子集合并成一个集合

1. 初始化

集合就是一些具有相同特征的元素构成的圈子，然后用其中某个元素作为整个圈子的代表。

![](https://img-blog.csdnimg.cn/20200612154913980.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpY2hhbzQxMA==,size_16,color_FFFFFF,t_70#pic_center)

如上图所示：初始的时候每个元素各自构成一个集合，自己为自己代言。

2. 合并

可以用树形结构来表示整个集合，每个元素代表一个节点，用根节点代表整个集合。合并两个元素就是对两个元素所在的集合进行合并，即直接把一个集合的根节点作为另外一个集合根节点的子节点。

![](https://img-blog.csdnimg.cn/20200612155154457.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpY2hhbzQxMA==,size_16,color_FFFFFF,t_70#pic_center)

如上图所示：元素1 和元素2 合并的结果就是，合并元素1、元素2 所在的集合，元素1为所属集合的根节点。

3. 查找

查找就是查询某个元素所在集合，即查找所在集合的根节点。

![](https://img-blog.csdnimg.cn/20200612155259462.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpY2hhbzQxMA==,size_16,color_FFFFFF,t_70#pic_center)

如上图所示，查找4所在集合的根节点，即对树形结构进行遍历逐层找父节点。

4. 路径优化

一个集合对应一个树形结构，但是我们并不真的关注谁是谁的父节点，而是对树形结构整体进行关注。因此我们可以把查找路径上的所有节点都直接和根节点相连，以缩短下次查找路径，如下图所示：

![](https://img-blog.csdnimg.cn/20200612155403425.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpY2hhbzQxMA==,size_16,color_FFFFFF,t_70#pic_center)

## 二．并查集的实现

1. 概述

对于每个元素仅需要记录其父节点即可，因此可以使用map来记录所有节点以及其父节点。
并查集的查找和合并的时间复杂度可以认为是O(1)。

2. 代码示例

```c++
class unionfind
{
private:
    unordered_map<int,int> root_map;//节点id—父节点id
public:
    //元素初始化
    void addnode(int i)
    {
        if( root_map.find(i) == root_map.end())
        {
            root_map[i] = i;//初始化父节点为自身id
        }
    }
    //判断x、y是否属于同一集合，判断其祖先节点是否相等
    bool checkxy( int x,int y)
    {
        x = getroot(x);//找x祖先节点
        y = getroot(y);//找y祖先节点
        if( x== y)
        {
            return true;
        }
        return false;
    }
    //合并元素x、y
    void mergexy( int x,int y)
    {
        x = getroot(x);//找x祖先节点
        y = getroot(y);//找y祖先节点
        if( x== y)//祖先相同则属于同一集合
        {
            return;
        }
        else
        {
            root_map[y] = x;//修改y的祖先节点为x
            return ;
        }
    }
    //查询元素祖先节点
    int getroot( int i)
    {
        if( root_map[i] == i)
        {
            return i;
        }
        else
        {
            root_map[i] = getroot(root_map[i]);//路径压缩
            return root_map[i];
        }
    }
};

```

## 三．应用举例

> 给定一个未排序的整数数组，找出最长连续序列的长度，要求算法的时间复杂度为 *O(n)*。
> **示例:**
> 输入: [100, 4, 200, 1, 3, 2]
> 输出: 4
> 解释: 最长连续序列是 [1, 2, 3, 4]。它的长度为 4。
> 来源：力扣（LeetCode）第128题

 本题可以采用并查集来实现，首先初始化所有元素父节点为自身，然后遍历数组中每个元素，查找比其值大1的元素是否存在，若存在则两个元素合并，最终比较合并后每个集合元素数量即可。

```c++
class Solution {
public:
    unordered_map<int,int> father_map;//节点-父节点
    unordered_map<int,int> child_count;// 节点-子节点个数
    int longestConsecutive(vector<int>& nums) {
        int res = 1;
        if( nums.size() == 0) return 0;
        for(int i = 0;i < nums.size();i++)
        {
            father_map[nums[i]] = nums[i];
            child_count[nums[i]] = 1;
        }
        for(int i = 0;i < nums.size();i++)
        {
             if( father_map.find(nums[i]+1) != father_map.end())
             {
                 res = max (res,mergexy(nums[i],nums[i]+1));
             }
        }
        return res;
    }
    int getfather(int i)
    {
        if( father_map[i] == i)
        {
            return i;
        }
        else
        {
            father_map[i] = getfather(father_map[i]);
            return father_map[i] ;
        }
    }
    int mergexy(int x,int y)
    {
        x = getfather(x);
        y = getfather(y);
        if( x == y )
        {
            return child_count[x];
        }
        else
        {
            father_map[y] = x;
            child_count[x] +=child_count[y];
            return child_count[x];
        }
    }
};
```

