---
layout: post
title: Leetcode贪心算法总结学习笔记
date: 2022-09-27
author: lau
tags: [Leetcode, Blog]
comments: true
toc: false
pinned: false

---

Leetcode贪心算法总结学习笔记。

<!-- more -->

## 概述

**贪心算法（又称贪婪算法）**是指，在对问题求解时，总是做出在当前看来是最好的选择。也就是说，不从整体最优上加以考虑，他所做出的是在某种意义上的局部最优解。

贪心算法不是对所有问题都能得到整体最优解，关键是贪心策略的选择，选择的贪心策略必须具备无后效性，即**某个状态以前的过程不会影响以后的状态**，只与当前状态有关。

### 基本思路

1. 建立数学模型来描述问题；
2. 把求解的问题分成若干个子问题；
3. 对每一子问题求解，得到子问题的局部最优解；
4. 把子问题的解局部最优解合成原来解问题的一个解。

### 应用案例

#### 活动安排问题

设有$n$个活动的集合$E=\{1,2,\cdots,n\}$，其中每个活动都要求使用同一资源，如演讲会场等，而在同一时间内只有一个活动能使用这一资源。每个活动$i$都有一个要求使用该资源的起始时间$s_i$和一个结束时间$f_i$,且$s_i < f_i$ 。要求设计程序，使得安排的活动最多。

| i      |  1   |  2   |  3   |  4   |  5   |  6   |  7   |  8   |  9   |  10  |
| ------ | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| Starti |  1   |  3   |  0   |  5   |  3   |  5   |  6   |  8   |  8   |  2   |
| Endi   |  4   |  5   |  6   |  7   |  8   |  9   |  10  |  11  |  12  |  13  |

**解题思路：**

题目的要求是安排的活动最多，故活动结束地早，那么安排的活动可以越多，对活动按照结束时间排序，然后遍历：若当前活动开始时间大于上一次活动的结束时间，则将其加入到队列中来。

#### [55. 跳跃游戏](https://leetcode-cn.com/problems/jump-game/)

> 给定一个非负整数数组 nums ，你最初位于数组的 第一个下标 。
>
> 数组中的每个元素代表你在该位置可以跳跃的最大长度。
>
> 判断你是否能够到达最后一个下标。
>
> 示例 1：
>
> 输入：nums = [2,3,1,1,4]
> 输出：true
> 解释：可以先跳 1 步，从下标 0 到达下标 1, 然后再从下标 1 跳 3 步到达最后一个下标。

**解题思路：**

这道算法题作为引入，主要体现了贪心算法的最直接的特点——“贪婪”，每一步都选择都选择从当前所在位置出发可以到达的最远距离，如给出一组数据list=[2,3,1,1,4]，对于位置0，list[0]=2,即使可以选择跳0,1,2步，但是我们“贪心的”只关注最远的mostlong的值，遍历整个数组，循环用mostlong=max(mostlong,i+nums[i]);更新mostlong的值，直到找到可以达到的最右端的位置。

```c++
class Solution {
public:
    bool canJump(vector<int>& nums) {
        int max_loc = 0;//最远可以到达的位置
        for(int i = 0; i < nums.size(); i++){
            //当前能达到的最远距离大于max_loc，则可以直接返回
            if(i + nums[i] >= nums.size() - 1)
                return true;
            //能达到的最远距离大于max_loc，则更新max_loc
            if(i + nums[i] > max_loc)
                max_loc = i + nums[i];
            if(max_loc == i)
                return false;
        }
        return false;
    }
};
```

#### **[53. 最大子序和](https://leetcode-cn.com/problems/maximum-subarray/)**

> 给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。
>
> 示例 1：
>
> 输入：nums = [-2,1,-3,4,-1,2,1,-5,4]
> 输出：6
> 解释：连续子数组 [4,-1,2,1] 的和最大，为 6 。

**解题思路：**

$ ans$表示以当前元素结尾的最大连续子段和，遍历数组时，实时更新维护最大值$max$，同时我们应该注意的是，若$ans<0$，则应该抛弃$ans$的累计值（$ans<0$的话，会导致后续计算出来的值肯定小于当前元素，应该抛弃），重新计数。

```c++
#include "bits/stdc++.h"
 
using namespace std;
 
class Solution {
public:
    int maxSubArray_greedy(vector<int>& nums) {
        int ans=0, max = -1000000;
        for(int i = 0; i < nums.size(); i++){
            if(ans <= 0){
                ans = nums[i];
            }
            else
                ans += nums[i];
            max = max > ans ? max : ans;
        }
        return max;
    }
};
```

#### [56. 合并区间](https://leetcode-cn.com/problems/merge-intervals/)

> 以数组 intervals 表示若干个区间的集合，其中单个区间为 intervals[i] = [starti, endi] 。请你合并所有重叠的区间，并返回一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间。
>
> 示例 1：
>
> 输入：intervals = [[1,3],[2,6],[8,10],[15,18]]
> 输出：[[1,6],[8,10],[15,18]]
> 解释：区间 [1,3] 和 [2,6] 重叠, 将它们合并为 [1,6].
> 示例 2：
>
> 输入：intervals = [[1,4],[4,5]]
> 输出：[[1,5]]
> 解释：区间 [1,4] 和 [4,5] 可被视为重叠区间。

```c++
class Solution {
public:
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        if(intervals.size() == 0)
            return {};
        //按照begin排序，再逐个遍历合并
        sort(intervals.begin(), intervals.end());
        vector<vector<int>> ans;
        for(int i = 0; i < intervals.size(); i++){
            int l = intervals[i][0], r = intervals[i][1];
            if(!ans.size() || ans.back()[1] < l){
                ans.push_back({l, r});
            }else{
                ans.back()[1] = max(ans.back()[1], r);
            }
        }
        return ans;
    }
};
```

#### [122. 买卖股票的最佳时机 II](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)

> 给你一个整数数组 prices ，其中 prices[i] 表示某支股票第 i 天的价格。
>
> 在每一天，你可以决定是否购买和/或出售股票。你在任何时候 最多 只能持有 一股 股票。你也可以先购买，然后在 同一天 出售。
>
> 返回 你能获得的 最大 利润 。
>
> 示例 1：
>
> 输入：prices = [7,1,5,3,6,4]
> 输出：7
> 解释：在第 2 天（股票价格 = 1）的时候买入，在第 3 天（股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5 - 1 = 4 。
>      随后，在第 4 天（股票价格 = 3）的时候买入，在第 5 天（股票价格 = 6）的时候卖出, 这笔交易所能获得利润 = 6 - 3 = 3 。
>      总利润为 4 + 3 = 7 。
> 示例 2：
>
> 输入：prices = [1,2,3,4,5]
> 输出：4
> 解释：在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5 - 1 = 4 。
>      总利润为 4 。
> 示例 3：
>
> 输入：prices = [7,6,4,3,1]
> 输出：0
> 解释：在这种情况下, 交易无法获得正利润，所以不参与交易可以获得最大利润，最大利润为 0 。

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        //在局部最低买入，在局部最高卖出
        //实际上是在所有涨价期间买入卖出即可
        int sum = 0;
        for(int i = 1; i < prices.size(); i++){
            if(prices[i] > prices[i-1])
                sum += (prices[i] - prices[i-1]);
        }
        return sum;
    }
};
```

#### [53. 最大子数组和](https://leetcode.cn/problems/maximum-subarray/)

> 给你一个整数数组 nums ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。
>
> 子数组 是数组中的一个连续部分。
>
> 示例 1：
>
> 输入：nums = [-2,1,-3,4,-1,2,1,-5,4]
> 输出：6
> 解释：连续子数组 [4,-1,2,1] 的和最大，为 6 。
> 示例 2：
>
> 输入：nums = [1]
> 输出：1
> 示例 3：
>
> 输入：nums = [5,4,-1,7,8]
> 输出：23

```c++
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        //记录到目前为止的最大和(保证大于0，否则清0)
        int pre_sum = 0, ans = INT_MIN;
        for(int i = 0; i < nums.size(); i++){
            //先更新连续子数组的最大和
            ans = max(ans, pre_sum + nums[i]);
            //再更新pre_sum
            if(pre_sum + nums[i] > 0)
                pre_sum += nums[i];
            else
                pre_sum = 0;
        }
        return ans;
    }
};
```

## 参考文献

[1] **https://blog.csdn.net/weixin_48994268/article/details/123121661**

[2] https://blog.csdn.net/qq_41648804/article/details/119836763

[3] https://blog.csdn.net/poolooloo/article/details/125918763