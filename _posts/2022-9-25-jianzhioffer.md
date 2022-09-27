---
layout: post
title: 剑指offer刷题总结学习笔记
date: 2022-09-26
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
|      ⭐       | [剑指 Offer 16. 数值的整数次方](https://leetcode.cn/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/) |   中等   |                           快速幂。                           |
|      ⭐       | [剑指 Offer 17. 打印从1到最大的n位数](https://leetcode.cn/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/) |   简单   |                递归造字符串，每层递归造一位。                |
|      ⭐       | [剑指 Offer 18. 删除链表的节点](https://leetcode.cn/problems/shan-chu-lian-biao-de-jie-dian-lcof/) |   简单   | 双指针，找到后前一个连上后一个（原题中是给了要删除节点的指针`node`，这时应该是`node.val = node.next.val;` `node.next = node.next.next;`）。 |
|      ⭐       | [剑指 Offer 19. 正则表达式匹配](https://leetcode.cn/problems/zheng-ze-biao-da-shi-pi-pei-lcof/) |   困难   | 动态规划，详细解法见[动态规划题目汇总（持续更新）](https://blog.csdn.net/weixin_43954951/article/details/124955963?spm=1001.2014.3001.5501)。 |
|      ⭐       | [剑指 Offer 20. 表示数值的字符串](https://leetcode.cn/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/) |   中等   | 用三个辅助函数`hasInteger()`、`hasDecimal()`和`atLeastOneNumber()`。 |
|      ⭐       | [剑指 Offer 21. 调整数组顺序使奇数位于偶数前面](https://leetcode.cn/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/) |   简单   | **1.** 左右双指针，类似快排中的`partition`函数。<br/>**2.** 快慢双指针，快指针碰到奇数就跟慢指针互换，快指针每次都加一，慢指针换过才加一。 |
|     ⭐⭐⭐      | [剑指 Offer 22. 链表中倒数第k个节点](https://leetcode.cn/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/) | **简单** | 双指针，第一个指针出发 k 步时第二个指针出发，第一个指针到结尾时返回第二个指针。 |
|    ⭐⭐⭐⭐⭐     | [剑指 Offer 24. 反转链表](https://leetcode.cn/problems/fan-zhuan-lian-biao-lcof/) | **简单** | **1.** 迭代，`pre`、`mid`和`post`三个指针，循环中每次让`mid.next = pre;`并让三个指针依次后移一位。<br/>**2.** 递归，对于每个节点`node`，递归解决后面的节点，然后让`node.next.next = node;`再让`node.next = null;`。 |
|    ⭐⭐⭐⭐⭐     | [ 剑指 Offer 25. 合并两个排序的链表](https://leetcode.cn/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/) |   简单   | 1. 迭代，pre指针指向已合并的链表的尾巴，p1和p2比谁小，谁小谁被pre连上，然后小的指针前进，pre也前进。<br/>2. 递归，递归函数的功能是将两个链表合并并返回头节点，内部先选出小的指针，将小指针的next和大指针放入递归函数进行递归，然后让小指针的next指向递归函数的返回值。<br/> |
|      ⭐       | [剑指 Offer 26. 树的子结构](https://leetcode.cn/problems/shu-de-zi-jie-gou-lcof/) |   中等   | 先实现判断“是不是从根开始的子结构”的递归函数，再用它实现判断“是不是子结构”的递归函数。 |
|     ⭐⭐⭐      | [剑指 Offer 27. 二叉树的镜像](https://leetcode.cn/problems/er-cha-shu-de-jing-xiang-lcof/) |   简单   | **1.** 迭代，用栈或队列遍历所有节点，每个节点都对调左右子树。<br/>**2.** 递归，先递归翻转左右子树，再将左右子树对调。<br/>**3.** 前中后序递归遍历，每个节点都对调左右子树。 |
|     ⭐⭐⭐      | [剑指 Offer 28. 对称的二叉树](https://leetcode.cn/problems/dui-cheng-de-er-cha-shu-lcof/) |   简单   | 1. 迭代，用一个队列，一对一对的装，每次从队列中取两个节点a、b来比，若它们的值相同，则将a.left和b.right一对装入队列，再将a.right和b.left一对装入队列，继续循环，若任何一对节点的值不同则返回false。<br/>2. 递归，两个指针root1、root2都从根节点出发开始递归，递归函数的功能是判断两棵树是不是对称，若根节点值相同且root1的左子树与root2的右子树对称、root1的右子树与root2的左子树对称，则返回true。<br/> |
|    ⭐⭐⭐⭐⭐     | [剑指 Offer 29. 顺时针打印矩阵](https://leetcode.cn/problems/shun-shi-zhen-da-yin-ju-zhen-lcof/) |   简单   | **1.** 按层模拟，每个边框执行完都让相应边界收缩一层，若收缩后超过了对面的边界则结束循环。<br/>**2.** 旋转模拟，需要一个额外数组记录访问过的位置。 |
|     ⭐⭐⭐      | [ 剑指 Offer 30. 包含min函数的栈](https://leetcode.cn/problems/bao-han-minhan-shu-de-zhan-lcof/) |   简单   | 加一个辅助栈`minStack`用于存储最小值。每当入栈的元素小于等于`minStack`栈顶的元素，就也将其入`minStack`栈；每当出栈的元素等于`minStack`栈顶的元素，就也将`minStack`出栈。 |
|      ⭐       | [剑指 Offer 31. 栈的压入、弹出序列](https://leetcode.cn/problems/zhan-de-ya-ru-dan-chu-xu-lie-lcof/) |   中等   | 1. 指针模拟，使用三个指针，分别指向下一个要入栈的元素、下一个要出栈的元素和栈顶元素，并用Integer.MIN_VALUE标记已出栈的元素，详细解法见栈的压入、弹出序列 不用栈模拟的解法 空间复杂度$O(1)$。<br/>2. 栈模拟，每次入栈都循环判断栈顶元素要不要出栈，直到栈顶元素不出栈或栈为空。<br/> |
|      ⭐       | [剑指 Offer 32 - I. 从上到下打印二叉树](https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/) |   中等   |                   用队列实现广度优先遍历。                   |
|    ⭐⭐⭐⭐⭐     | [剑指 Offer 32 - II. 从上到下打印二叉树 II](https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-ii-lcof/) |   简单   | **1.** 用队列实现广度优先遍历，外层循环中先记队列当前长度`length`，然后进行`length`次内层循环以遍历一层。<br/>**2.** 递归，递归函数有个`layer`参数，使得每个节点可以放入第`layer`个`List`。 |
|      ⭐       | [剑指 Offer 32 - III. 从上到下打印二叉树 III](https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/) |   中等   | 1. 用队列实现广度优先遍历，外层循环中先记队列当前长度length，然后进行length次内层循环以遍历一层的节点并依次放入双端队列中，奇数层从头进、偶数层从尾进，然后将双端队列转List并放入resultLists。<br/>2. 递归，递归函数有个layer参数，使得每个节点可以放入第layer个List，再加一个leftFirst参数让递归函数只在奇数层或偶数层添加结果。分别针对奇数层和偶数层使用两次递归函数即可得到答案。<br/> |
|      ⭐       | [剑指 Offer 33. 二叉搜索树的后序遍历序列](https://leetcode.cn/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/) |   中等   | 将序列反转，然后用一个栈模拟二叉搜索树的构建过程，详细解法见[二叉搜索树的后序遍历序列（辅助栈 最清晰易懂的可视化讲解）](https://blog.csdn.net/weixin_43954951/article/details/124697359?spm=1001.2014.3001.5501)。 |
|     ⭐⭐⭐      | [剑指 Offer 34. 二叉树中和为某一值的路径](https://leetcode.cn/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/) |   中等   | 前序遍历，将每个节点加入路径，若叶节点处`target`为 0 则添加路径，每个节点完成时从路径中移除并还原`target`。 |
|      ⭐⭐      | [剑指 Offer 35. 复杂链表的复制](https://leetcode.cn/problems/fu-za-lian-biao-de-fu-zhi-lcof/) |   中等   | **分三步完成：**<br/>**（1）** 复制，在原链表中复制每个节点使其长度变为两倍。<br/>**（2）** 连接，对于每个节点`node`，`node.next.random = node.random.next;`。<br/>**（3）** 脱钩，遍历链表让原链表与新链表脱钩。 |
|      ⭐⭐      | [剑指 Offer 36. 二叉搜索树与双向链表](https://leetcode.cn/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/) |   中等   | 中序遍历，也就是按升序遍历二叉搜索树，使用一个指针`preNode`指向上一个遍历到的节点（即当前节点的左指针该连的节点），在每个节点让该节点与`preNode`互连，然后让`preNode`指向该节点。 |
|      ⭐       | [剑指 Offer 37. 序列化二叉树](https://leetcode.cn/problems/xu-lie-hua-er-cha-shu-lcof/) |   困难   | **1.** 用括号，序列化是“(左子树)根节点(右子树)”的形式，反序列化是先找到根节点位置然后把左右子树递归构建。<br/>**2.** 用逗号，序列化是前序遍历序列且空节点用空格表示，反序列化是将序列转为链表并以先根顺序递归构建。 |
|      ⭐       | [剑指 Offer 38. 字符串的排列](https://leetcode.cn/problems/zi-fu-chuan-de-pai-lie-lcof/) |   中等   | **1.** 用下一个排列法，先升序排列，然后不断计算下一个排列，直到整个序列变为降序。下一个排列的详细解法见[下一个排列（双指针）](https://blog.csdn.net/weixin_43954951/article/details/125791760?spm=1001.2014.3001.5501)。<br/>**2.** 回溯法，空间复杂度高于下一个排列法。 |
|     ⭐⭐⭐      | [剑指 Offer 39. 数组中出现次数超过一半的数字](https://leetcode.cn/problems/shu-zu-zhong-chu-xian-ci-shu-chao-guo-yi-ban-de-shu-zi-lcof/) |   简单   | 1. Boyer-Moore 投票算法，遍历投票候选人，还是他就加一，不是他就减一，票数是 $0$ 了就换下一个做候选人，遍历结束后候选人就是出现次数超过一半的数字。<br/>2. 哈希表，需要 $O(n)$的额外空间。<br/> |
|      ⭐       | [剑指 Offer 41. 数据流中的中位数](https://leetcode.cn/problems/shu-ju-liu-zhong-de-zhong-wei-shu-lcof/) |   困难   | 用两个堆，一个最大堆一个最小堆，始终维持两个堆的大小差不超过 1 。 |
|    ⭐⭐⭐⭐⭐     | [剑指 Offer 42. 连续子数组的最大和](https://leetcode.cn/problems/lian-xu-zi-shu-zu-de-zui-da-he-lcof/) |   简单   | 1. 线段树，递归分治整个数组（类似二叉树形式），对每个子数组（树的节点）维护四个值，即总和、左起最大总和、右起最大总和以及最大子区间总和，以后根顺序计算每个节点的四个值，根节点的最大子区间总和即为答案。该方法虽然空间复杂度更高，还用了递归，但是只要维护成线段树，即可在 $O(\log n)$的时间内求得任意区间的答案。<br/>2. 动态规划，详细解法见动态规划题目汇总（持续更新）。<br/> |
|      ⭐       | [剑指 Offer 43. 1～n 整数中 1 出现的次数](https://leetcode.cn/problems/1nzheng-shu-zhong-1chu-xian-de-ci-shu-lcof/) |   困难   | 逐位计算，把整个数字分为高位数字（highNumber）、当前位数字（digit）和低位数字（lowNumber）三个部分，当前位从个位开始向左遍历。维护一个变量powerOfTen，在第$i$位时其值为$10^i$  ，其意义为当前位前面的数字翻一次时当前位会转过多少次 $1$ 。对每个位分三种情况讨论，若`digit == 0`，则`sum += highNumber * powerOfTen;`，若`digit == 1`则`sum += highNumber * powerOfTen + lowNumber + 1;`，若`digit > 1`则`sum += (highNumber + 1) * powerOfTen;`。<br/> |
|      ⭐       | [剑指 Offer 44. 数字序列中某一位的数字](https://leetcode.cn/problems/shu-zi-xu-lie-zhong-mou-yi-wei-de-shu-zi-lcof/) |   中等   | 设$n$为$d$位数，先求前$d-1$位的所有数字共占的位数$s$，然后计算第$n$位所在的数字并确定是该数字的哪一位。 |
|      ⭐       | [剑指 Offer 45. 把数组排成最小的数](https://leetcode.cn/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/) |   中等   | **1.** 用快速排序，比较方式为两个字符串拼接谁在前面让拼接结果更大谁就更大。<br/>**2.** 用内置排序函数，比较方式为两个字符串拼接谁在前面让拼接结果更大谁就更大。 |
|      ⭐       | [剑指 Offer 46. 把数字翻译成字符串](https://leetcode.cn/problems/ba-shu-zi-fan-yi-cheng-zi-fu-chuan-lcof/) |   中等   | 动态规划，详细解法见[动态规划题目汇总（持续更新）](https://blog.csdn.net/weixin_43954951/article/details/124955963?spm=1001.2014.3001.5501)。 |
|      ⭐       | [剑指 Offer 47. 礼物的最大价值](https://leetcode.cn/problems/li-wu-de-zui-da-jie-zhi-lcof/) |   中等   | 动态规划，详细解法见[动态规划题目汇总（持续更新）](https://blog.csdn.net/weixin_43954951/article/details/124955963?spm=1001.2014.3001.5501)。 |
|    ⭐⭐⭐⭐⭐     | [剑指 Offer 48. 最长不含重复字符的子字符串](https://leetcode.cn/problems/zui-chang-bu-han-zhong-fu-zi-fu-de-zi-zi-fu-chuan-lcof/) |   中等   | 动态规划，详细解法见[动态规划题目汇总（持续更新）](https://blog.csdn.net/weixin_43954951/article/details/124955963?spm=1001.2014.3001.5501)。 |
|      ⭐       | [剑指 Offer 49. 丑数](https://leetcode.cn/problems/chou-shu-lcof/) |   中等   | 动态规划 + 三个指针，详细解法见[动态规划题目汇总（持续更新）](https://blog.csdn.net/weixin_43954951/article/details/124955963?spm=1001.2014.3001.5501)。 |
|      ⭐       | [ 剑指 Offer 50. 第一个只出现一次的字符](https://leetcode.cn/problems/di-yi-ge-zhi-chu-xian-yi-ci-de-zi-fu-lcof/) | **简单** | **1.** `LinkedHashMap`，出现过一次存`true`，两次存`false`，最后返回第一个为`true`的元素即可。**2.** `HashMap + Queue`，`HashMap`中出现过一次存`true`，两次存`false`，每次出现第二次时将`Queue`前端所有在`HashMap`中为`false`的元素`poll()`，最后返回队列最前端的元素。 |
|      ⭐       | [剑指 Offer 51. 数组中的逆序对](https://leetcode.cn/problems/shu-zu-zhong-de-ni-xu-dui-lcof/) |   困难   | 用归并排序对数组进行排序，在归并过程中每当左子数组中元素要被合并时，看看右子数组已经被合并了多少个元素，然后逆序对总数加上该数值。 |
|    ⭐⭐⭐⭐⭐     | [剑指 Offer 52. 两个链表的第一个公共节点](https://leetcode.cn/problems/liang-ge-lian-biao-de-di-yi-ge-gong-gong-jie-dian-lcof/) |   简单   | 双指针，分别从两个链表的头节点出发，到终点了就回到另一个链表的头节点，两个指针相遇点即为两个链表的交点，若两个指针同时为`null`则两个链表无交点。 |
|     ⭐⭐⭐      | [剑指 Offer 53 - I. 在排序数组中查找数字 I](https://leetcode.cn/problems/zai-pai-xu-shu-zu-zhong-cha-zhao-shu-zi-lcof/) |   简单   | 用二分查找的变体算法（二分查找目标值应该插入的位置）找到目标数字应该插入的位置和目标数字减一应该插入的位置，二者相减即可得到目标数字出现的次数，二分查找的变体算法的详细信息见[Java二分查找模板](https://blog.csdn.net/weixin_43954951/article/details/125086266?spm=1001.2014.3001.5501)。 |
|      ⭐       | [ 剑指 Offer 53 - II. 0～n-1中缺失的数字](https://leetcode.cn/problems/que-shi-de-shu-zi-lcof/) |   简单   | 用二分查找的变体算法，中间元素等于其下标则向右找，中间元素大于其下标则向左找，最后返回左指针。 |
|      ⭐       | [剑指 Offer 54. 二叉搜索树的第k大节点](https://leetcode.cn/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/) |   简单   |    用先右后左的中序遍历，维护全局变量用于计数和保存结果。    |
|     ⭐⭐⭐      | [剑指 Offer 55 - I. 二叉树的深度](https://leetcode.cn/problems/er-cha-shu-de-shen-du-lcof/) |   简单   | **1.** 深度优先遍历，二叉树的最大深度为其左右子树的最大深度中的较大者加一。<br/>**2.** 广度优先遍历，用队列进行按层遍历（双层循环，外循环遍历每层，内循环先获取当前队列长度，然后遍历当前层的所有节点），从而计算深度 |
|     ⭐⭐⭐      | [ 剑指 Offer 55 - II. 平衡二叉树](https://leetcode.cn/problems/ping-heng-er-cha-shu-lcof/) |   简单   | 用一个递归函数`depthOfBalancedTree()`，该函数判断一棵树是不是平衡二叉树，若是则返回其最大高度，若不是则返回 -1。 |
|      ⭐       | [剑指 Offer 56 - I. 数组中数字出现的次数](https://leetcode.cn/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/) |   中等   | 利用数字同本身异或结果为0的特性，先将所有数字异或，结果等于两个目标数字的异或结果，然后随便取这个结果中为 1 的一个二进制位（可以去掉最后面的一位 1 后再与原数字异或），对所有数字进行分组，这位是 1 的一起异或一遍，这位是 0 的一起异或一遍，两组的结果即分别为两个目标数字。 |
|      ⭐       | [剑指 Offer 56 - II. 数组中数字出现的次数 II](https://leetcode.cn/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-ii-lcof/) |   中等   | 对于所有数字，统计每个二进制位中出现过 1 的次数，对 3 求余不为 0 的位组合在一起即为目标数字。 |
|      ⭐       | [剑指 Offer 57. 和为s的两个数字](https://leetcode.cn/problems/he-wei-sde-liang-ge-shu-zi-lcof/) | **简单** | 双指针，一个从左出发一个从右出发，二者的和太小就让左指针右移，太大就让右指针左移，直到二者的和正好是 s 为止。 |
|      ⭐       | [剑指 Offer 57 - II. 和为s的连续正数序列](https://leetcode.cn/problems/he-wei-sde-lian-xu-zheng-shu-xu-lie-lcof/) |   简单   | **1.** 双指针，用变量`sum`保存当前左右指针之间所有数的总和，`sum < s`就让左指针右移，`sum > s`就让右指针左移，`sum == s`就保存左右指针之间的数字。<br />**2.** 滑动窗口，用一个`List`作为窗口，用变量`sum`保存当前窗口中的数的总和，移动方法与双指针法类似（实际上双指针法就是用两个指针模拟滑动窗口，从而减少空间开销） |
|     ⭐⭐⭐      | [剑指 Offer 58 - I. 翻转单词顺序](https://leetcode.cn/problems/fan-zhuan-dan-ci-shun-xu-lcof/) |   简单   | **1.** 双指针，一个指向单词头一个指向单词尾，先两个指针一起跳过空格走到一个单词的尾部，然后头指针自己向左走到单词头部，把该单词连接到结果字符串上，尾指针跳至头指针位置，重复上述过程直至头指针走完整个字符串。<br />**2.** 使用`split`函数，按空格分割字符串得到字符串数组，倒序遍历数组，凡是长度大于 0 的字符串都连接到结果字符串上。 |
|      ⭐       | [剑指 Offer 58 - II. 左旋转字符串](https://leetcode.cn/problems/zuo-xuan-zhuan-zi-fu-chuan-lcof/) |   简单   |           分两段遍历字符串并添加到结果字符串即可。           |
|     ⭐⭐⭐      | [剑指 Offer 59 - I. 滑动窗口的最大值](https://leetcode.cn/problems/hua-dong-chuang-kou-de-zui-da-zhi-lcof/) |   困难   | 用单调队列模拟滑动窗口，利用双端队列（设左为头右为尾）实现，其特点是每当有新的元素要从尾部进入时，让尾部所有比新元素小的元素依次出队，然后新元素才从尾部入队。因为只要这个新元素还在窗口中，那些比它小的尾部元素就不可能是最大值，同时这个新元素又一定比那些比它小的尾部元素出队晚，所以那些比它小的尾部元素已经没又存在的必要了。在窗口向右滑动时，若左侧刚刚“离开”窗口的元素等于队列头部的元素，则说明该元素确实离开了窗口，于是将其出队。 |
|      ⭐       | [ 剑指 Offer 59 - II. 队列的最大值](https://leetcode.cn/problems/dui-lie-de-zui-da-zhi-lcof/) |   中等   | 维护一个单调队列，用双端队列实现。当有元素入队时，让单调队列尾部所有比新元素小的元素依次出队，然后让新元素也进入单调队列；当有元素出队时，若该元素等于单调队列头部的元素，则也让单调队列头部的元素出队。 |
|      ⭐       | [剑指 Offer 60. n个骰子的点数](https://leetcode.cn/problems/nge-tou-zi-de-dian-shu-lcof/) | **中等** | 动态规划，详细解法见[动态规划题目汇总（持续更新）](https://blog.csdn.net/weixin_43954951/article/details/124955963?spm=1001.2014.3001.5501)。 |
|      ⭐       | [剑指 Offer 61. 扑克牌中的顺子](https://leetcode.cn/problems/bu-ke-pai-zhong-de-shun-zi-lcof/) | **简单** | 遍历所有牌并用`HashSet`存储每张不为 0 的牌（用于检查是否有重复的牌，若出现重复则直接返回`false`），同时统计出最大值与最小值，最后最大值与最小值的差小于牌的数量，就说明可以组成顺子，否则就不能。 |
|      ⭐       | [剑指 Offer 62. 圆圈中最后剩下的数字](https://leetcode.cn/problems/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-lcof/) |   简单   | 用约瑟夫环数学法，详细解法见[圆圈中最后剩下的数字（约瑟夫环问题）](https://blog.csdn.net/weixin_43954951/article/details/125685690?spm=1001.2014.3001.5501)。 |
|    ⭐⭐⭐⭐⭐     | [剑指 Offer 63. 股票的最大利润](https://leetcode.cn/problems/gu-piao-de-zui-da-li-run-lcof/) |   中等   | 动态规划，详细解法见[动态规划题目汇总（持续更新）](https://blog.csdn.net/weixin_43954951/article/details/124955963?spm=1001.2014.3001.5501)。 |
|      ⭐       | [剑指 Offer 64. 求1+2+…+n](https://leetcode.cn/problems/qiu-12n-lcof/) |   中等   |               递归，并利用`&&`或`||`终止递归。               |
|      ⭐       | [ 剑指 Offer 65. 不用加减乘除做加法](https://leetcode.cn/problems/bu-yong-jia-jian-cheng-chu-zuo-jia-fa-lcof/) |   简单   | 用二进制位运算模拟加法，两个二进制数的和等于它们不考虑进位的二进制和与进位的和，详细解法见[不用加减乘除做加法 从递归到迭代](https://blog.csdn.net/weixin_43954951/article/details/125729874?spm=1001.2014.3001.5501)。 |
|      ⭐       | [剑指 Offer 66. 构建乘积数组](https://leetcode.cn/problems/gou-jian-cheng-ji-shu-zu-lcof/) |   中等   | 用一个变量保存中间乘积，从左到右乘一遍，再从右到左乘一遍，即可得到答案。 |
|     ⭐⭐⭐      | [剑指 Offer 67. 把字符串转换成整数](https://leetcode.cn/problems/ba-zi-fu-chuan-zhuan-huan-cheng-zheng-shu-lcof/) |   中等   | 遍历判断，先去空格，再判断是否是数字，最后取出数字（取数字的过程中一旦当前数字已大于最大整数的十分之一或下一个数字小于 0 ，则说明数字超出了整型范围，马上返回题目要求的特殊值）。 |
|      ⭐       | [剑指 Offer 68 - I. 二叉搜索树的最近公共祖先](https://leetcode.cn/problems/er-cha-sou-suo-shu-de-zui-jin-gong-gong-zu-xian-lcof/) |   简单   | 类似二分查找，找到一个大于较小的目标节点且小于较大的目标节点的节点即是答案。 |
|    ⭐⭐⭐⭐⭐     | [剑指 Offer 68 - II. 二叉树的最近公共祖先](https://leetcode.cn/problems/er-cha-shu-de-zui-jin-gong-gong-zu-xian-lcof/) |   中等   | 1. 返回boolean的递归函数，函数的作用是判断输入的树中是否有两个目标节点中的一个或两个，用一个全局遍历保存最近公共祖先，当递归函数中左子树的结果与右子树的结果同时为true或一个子树的结果为true但根节点与一个目标节点相同时，将此时的根节点保存为最近公共祖先（一定只会保存一次，因为除了最近公共祖先不会有任何其他节点满足条件）。<br />**2.** 返回节点的递归函数，函数的作用是返回左子树中出现的目标节点或左子树中出现的目标节点，若左右子树中都存在目标节点或根节点与一个目标节点相同则返回根节点，若右子树中都不存在目标节点则返回`null`。 |



## 参考文献

[1] [《剑指offer》所有题目解法思路与出现频率汇总](https://blog.csdn.net/weixin_43954951/article/details/125821879?spm=1001.2101.3001.6650.4&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-4-125821879-blog-109862706.t0_edu_mix&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-4-125821879-blog-109862706.t0_edu_mix&utm_relevant_index=5)

[2] [剑指offer所有题目总结](https://blog.csdn.net/wszy1301/article/details/80910626?spm=1001.2101.3001.6650.10&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-10-80910626-blog-125821879.pc_relevant_multi_platform_featuressortv2dupreplace&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-10-80910626-blog-125821879.pc_relevant_multi_platform_featuressortv2dupreplace&utm_relevant_index=11)

[3] [万字长文！剑指offer全题解思路汇总](https://blog.csdn.net/Sheng_zhenzhen/article/details/107747644?spm=1001.2101.3001.6650.6&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-6-107747644-blog-125821879.pc_relevant_multi_platform_featuressortv2dupreplace&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-6-107747644-blog-125821879.pc_relevant_multi_platform_featuressortv2dupreplace&utm_relevant_index=7)



