---
layout: post
title: LeetCode刷题回溯算法相关知识笔记
date: 2022-08-10
author: lau
tags: [Leetcode,Archive]
comments: true
toc: false
pinned: false
---
LeetCode刷题回溯算法相关知识笔记。

<!-- more -->

## 回溯法概述
一般情况下，看到题目要求「所有可能的结果」，而不是「结果的个数」，我们就知道需要暴力搜索所有的可行解了，可以用「回溯法」。
「回溯法」实际上一个类似枚举的搜索尝试过程，主要是在搜索尝试过程中寻找问题的解，当发现已不满足求解条件时，就「回溯」返回，尝试别的路径。
回溯法是一种算法思想，而递归是一种编程方法，回溯法可以用递归来实现。
回溯法的整体思路是：搜索每一条路，每次回溯是对具体的一条路径而言的。对当前搜索路径下的的未探索区域进行搜索，则可能有两种情况：
- 当前未搜索区域满足结束条件，则保存当前路径并退出当前搜索；
- 当前未搜索区域需要继续搜索，则遍历当前所有可能的选择：如果该选择符合要求，则把当前选择加入当前的搜索路径中，并继续搜索新的未探索区域。
上面说的未搜索区域是指搜索某条路径时的未搜索区域，并不是全局的未搜索区域。
回溯法搜所有可行解的模板一般是这样的：
```c++
res = []
path = []
def backtrack(未探索区域, res, path): #参数写的时候确定
    if path 满足条件:
        res.add(path) # 深度拷贝
        # return  # 如果不用继续搜索需要 return
    for 选择 in 未探索区域当前可能的选择:
        if 当前选择符合要求:
            path.add(当前选择)
            backtrack(新的未探索区域, res, path)
            path.pop()
```
回溯算法可以解决：
- 组合问题
- 切割问题
- 子集问题
- 排列问题
- 棋盘问题
## 回溯算法理解
回溯法都可以抽象为一个多叉树，树的宽度为每个节点要处理的大小，一般用for循环来处理，树的高度为递归的层数，一般用递归函数来表示。
## 回溯算法例题
例题1：
给你一个整数数组 nums ，其中可能包含重复元素，请你返回该数组所有可能的子集（幂集）。
解集 不能 包含重复的子集。返回的解集中，子集可以按 任意顺序 排列。
```c++
class Solution {
public:
    vector<vector<int>> subsetsWithDup(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        vector<vector<int>> res;
        vector<int> path;
        backtrack(res, path, nums, 0);
        return res;
    }
    void backtrack(vector<vector<int>>& res, vector<int>& path, vector<int>& nums, int index) {
        if (index > nums.size()) return;
        res.push_back(path);
        for (int i = index; i < nums.size(); ++i) {
            if (i != index && nums[i] == nums[i - 1]) continue;
            path.push_back(nums[i]);
            backtrack(res, path, nums, i + 1);
            path.pop_back();
        }
    }
};
```
[算法学习随笔 7_回溯算法整理总结](https://blog.csdn.net/P_in_k/article/details/124541902)
