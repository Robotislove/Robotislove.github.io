---
layout: post
title: 剑指offer刷题总结学习笔记
date: 2022-09-25
author: lau
tags: [Leetcode, Blog]
comments: true
toc: false
pinned: false



---

剑指offer刷题总结学习笔记。

<!-- more -->

| **出现频率** |                           **题目**                           | **难度** |                           **思路**                           |
| :----------: | :----------------------------------------------------------: | :------: | :----------------------------------------------------------: |
|      ⭐       | [剑指 Offer 03. 数组中重复的数字](https://leetcode.cn/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/) | **简单** | 每个 $n$ 都放入坐标为$n$的位置，那个位置本来就是 $n$了就返回。 |
|     ⭐⭐⭐      | [剑指 Offer 04. 二维数组中的查找](https://leetcode.cn/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/) | **中等** |        从右上角（或左下角）开始，小了往下，大了往左。        |
|      ⭐       | [ 剑指 Offer 05. 替换空格](https://leetcode.cn/problems/ti-huan-kong-ge-lcof/) |   简单   |                         C++插入删除                          |
|      ⭐       | [剑指 Offer 06. 从尾到头打印链表](https://leetcode.cn/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/) |   简单   | **1.** 先遍历算长度，再遍历存入数组。**2.** 循环用栈装。**3.** 递归用`List`从后往前装。 |
|     ⭐⭐⭐      | [剑指 Offer 07. 重建二叉树](https://leetcode.cn/problems/zhong-jian-er-cha-shu-lcof/) | **中等** | 前序序列第一个是根，用它分割中序序列，左右两半（左右子树的序列）递归处理。 |
|      ⭐       | [剑指 Offer 09. 用两个栈实现队列](https://leetcode.cn/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/) |   简单   | 一个`InStack`一个`OutStack`，要`offer`就进`InStack`，要`poll`就出`OutStack`，`OutStack`空了就把`InStack`全倒进`OutStack`。 |
|      ⭐       | [ 剑指 Offer 10- I. 斐波那契数列](https://leetcode.cn/problems/fei-bo-na-qi-shu-lie-lcof/) |   简单   | **1.** 矩阵快速幂（矩阵乘法中间结果`sum`用`long`，放进乘积矩阵前求余并转`int`）。**2.** 动态规划（不用`long`，每次`f[i]`都求余即可）。 |
|     ⭐⭐⭐⭐     | [剑指 Offer 10- II. 青蛙跳台阶问题](https://leetcode.cn/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/) | **简单** |                                                              |
|     ⭐⭐⭐      | [剑指 Offer 11. 旋转数组的最小数字](https://leetcode.cn/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/) |   简单   | 二分查找，`mid`跟`right`比。若用`left < right`做循环终止条件则`return left`或`return right`都一样，因为最后两个指针会重合；若用`left <= right`做循环终止条件则`return left`。 |
|      ⭐       | [剑指 Offer 12. 矩阵中的路径](https://leetcode.cn/problems/ju-zhen-zhong-de-lu-jing-lcof/) |   中等   | 回溯法，用一个递归函数判断以该格子里的字母打头的词存不存在，然后用这个递归函数遍历所有格子。 |
|      ⭐       | [剑指 Offer 13. 机器人的运动范围](https://leetcode.cn/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/) |   中等   | 递推，遍历每个格子，若该格子的左边或上边格子能到，这个格子才有可能到，然后再判断该格子的数位和是否满足条件。 |
|      ⭐       | [剑指 Offer 14- I. 剪绳子](https://leetcode.cn/problems/jian-sheng-zi-lcof/) |   中等   | **1.** 数学法 + 快速幂，尽量全剪成 3，最后一节分 0、1、2 三种情况处理，计算时用快速幂算法。<br/>**2.** 动态规划，详细解法见[动态规划题目汇总（持续更新）](https://blog.csdn.net/weixin_43954951/article/details/124955963?spm=1001.2014.3001.5501)。 |
|      ⭐       | [ 剑指 Offer 14- II. 剪绳子 II](https://leetcode.cn/problems/jian-sheng-zi-ii-lcof/) |   中等   | **需要求余的剪绳子**<br/>**1.** 数学法 + 快速幂，快速幂函数中全用`long`，有乘法就求余，返回值乘 3 后求余再转`int`。<br/>**2.** 动态规划，有乘法处求余，详细解法见[动态规划题目汇总（持续更新）](https://blog.csdn.net/weixin_43954951/article/details/124955963?spm=1001.2014.3001.5501)。 |
|      ⭐       | [剑指 Offer 15. 二进制中1的个数](https://leetcode.cn/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/) |   简单   |            `n & (n - 1)`可以去掉`n`最后一位的 1。            |

## 参考文献

[1] [《剑指offer》所有题目解法思路与出现频率汇总](https://blog.csdn.net/weixin_43954951/article/details/125821879?spm=1001.2101.3001.6650.4&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-4-125821879-blog-109862706.t0_edu_mix&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-4-125821879-blog-109862706.t0_edu_mix&utm_relevant_index=5)

