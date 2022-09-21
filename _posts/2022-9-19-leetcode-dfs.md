---
layout: post
title: LeetCode刷题DFS深度优先遍历使用总结学习笔记
date: 2022-08-28
author: lau
tags: [Leetcode, Blog]
comments: true
toc: false
pinned: false



---

 LeetCode刷题DFS深度优先遍历使用总结学习笔记。

<!-- more -->

## DFS算法思想

深度优先搜索就是在每一步对每一种可能的选择一条道走到底，然后再回过头尝试另外一种选择。

深度优先搜索的关键是要考虑“**当前这一步**”该如何做，至于“下一步”该怎么做和当前这一步的解决方法是一样的。在进行当前步的选择之前**要确定已经做出的选择**

**列表**，然后在**剩余可供选择的每一种可能进行遍历**，对于每一种选择将**选择结果以及选择状态代入下一步操作**，然后再次进行深度优先搜索。

DFS的实现考虑要以下几个问题即可：
    ①．边界范围：**即搜索终止条件，递归结束条件**。
    ②．可供选择的范围列表：**所有可供选择的范围列表**。
    ③．已做出的选择列表：**标记当前已经做出的选择**。

### DFS代码模版

```c++
void dfs(int cur_step) {
    // 判断边界
    // 尝试当前可供选择的每种可能
    for (int i = 0; i < maxCount; i++) {
        // 尝试每种选择
        // 选择结果代入下一步
        dfs(cut_step+1);
        //撤销当前选择，恢复状态
    }
}
```



## 应用实例

### 八皇后问题

### 全排列问题

#### 解题思路

进行数字的全排列即用给定的数字范围在数组的每个位置进行每种情况的尝试，每一步选择时的操作方法是一样的，即可以DFS进行实现。

由于存在重复的数字，即需要保证每个位置对数值相同的数字只做一次选择，可先将原数组进行排序，然后遍历时跳过相同的值即可。

```c++
class Solution {
public:
    vector<vector<int>> permuteUnique(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        vector<vector<int>> res; // 结果
        vector<int> vis = vector(nums.size(), 0); // 标记元素选择状态
        if (nums.size() == 0) return res;
        vector<int> cur_vector; // 当前进行中的排列
        dfs(res, nums, vis, cur_vector);
        return res;
    }

    void dfs(vector<vector<int>>& res, vector<int>& nums, vector<int>& vis, vector<int> cur_vector) {
        if (cur_vector.size() == nums.size()) {
            res.push_back(cur_vector);
            return;
        }

        for (int i = 0; i < nums.size(); i++) {
            if(vis[i] == 1) continue;
            int cur_value = nums[i];
            cur_vector.push_back(nums[i]);
            vis[i] = 1;
            dfs(res, nums, vis, cur_vector);
            cur_vector.pop_back();
            vis[i] = 0;
            while (i + 1 < nums.size() && nums[i + 1] == nums[i]){
                i++;
            }            
        }
    }
};
```

