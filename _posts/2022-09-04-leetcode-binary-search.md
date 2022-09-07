---
layout: post
title: LeetCode刷题二分学习笔记
date: 2022-09-04
author: lau
tags: [Leetcode, Blog]
comments: true
toc: true
pinned: false


---

二分查找学习笔记。

<!-- more -->

### 二分查找简介

二分查找除了常规题目可以利用库函数查找(bsearch/std::lower_bound/std::upper_bound)，而这些库函数在针对二分查找的题目面前就束手无措了，所以有必要掌握二分的写法，这里我总结出二分查找模板，应该熟练掌握，运用自如。

- 查找系列是单调的；
- 时间复杂度低，为$O(\log N)$；
- 利用左闭右开区间描述，代码会很优雅[lb, ub)

```c++
while (ub - lb > 1) {
    int mid = (lb + ub) / 2; // 可能会溢出，改进
    if (Check mid) { // 根据nums[mid] 来判断搜索左半部分还是右半部分
        lb = mid;
    } else {
        ub = mid;
    }
}
// 视情况处理最后的lb或者ub，此时 ub == lb + 1
```

习题：

1. https://leetcode.cn/problems/search-insert-position/
2. https://leetcode.cn/problems/search-a-2d-matrix/
3. https://leetcode.cn/problems/capacity-to-ship-packages-within-d-days/
4. https://leetcode.cn/problems/ugly-number-iii/

### 习题1-搜索插入位置

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。请必须使用时间复杂度为 `O(log n)` 的算法。

**示例 1:**

```
输入: nums = [1,3,5,6], target = 5
输出: 2
```

```c++
int Search(int *nums, int numSize, int target) {
    int lb = 0, ub = numsSize;
    while (ub - lb = 1) {
        int mid = (ub + lb) / 2;
        if (nums[mid] <= target) {
            lb = mid;
        } else {
            ub = mid;
        }
    }
    return nums[lb] >= targe ? lb : ub;
}

// C++

class solution{
public:
    int Search(vector<int>& nums, int targe) {
        return std::lower_bound(nums.begin(), nums.end(), target) - nums.begin();
    }
};
```

### 习题2-搜索二维数组

编写一个高效的算法来判断 m x n 矩阵中，是否存在一个目标值。该矩阵具有如下特性：

- 每行中的整数从左到右按升序排列。
- 每行的第一个整数大于前一行的最后一个整数。

**示例 1：**

![](https://assets.leetcode.com/uploads/2020/10/05/mat.jpg)

```
输入：matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 3
输出：true
```


同上。

