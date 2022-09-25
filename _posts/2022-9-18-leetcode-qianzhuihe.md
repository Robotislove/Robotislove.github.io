---
layout: post
title: LeetCode刷题前缀和总结学习笔记
date: 2022-09-18
author: lau
tags: [Leetcode, Blog]
comments: true
toc: false
pinned: false
---

LeetCode刷题前缀和使用总结学习笔记。

<!-- more -->

## 前缀和基础知识

1. 概念

   前缀和是一个数组的某项下标之前(包括该元素)的所有数组元素的和，前缀和是一种重要的预处理操作，可以降低查询的时间复杂度。

2. 一维前缀和

   一维前缀和主要用于使用$O(1)$的时间，计算出区间和：$nums[i]+nums[i+1]+…+nums[j]$.

   

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2xpemhpY2hhbzMxOCUyMC9QaWNnby9tYXN0ZXIvaW1nLzIwMjAwNTIzMTU0MjQwLmpwZw?x-oss-process=image/format,png#pic_center)

3. 二位前缀和

二维前缀和主要用于对一个矩阵，在$O(1)$的时间内计算出子矩阵的和。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2xpemhpY2hhbzMxOCUyMC9QaWNnby9tYXN0ZXIvaW1nLzIwMjAwNTIzMTU0MjE5LmpwZw?x-oss-process=image/format,png#pic_center)

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2xpemhpY2hhbzMxOCUyMC9QaWNnby9tYXN0ZXIvaW1nLzIwMjAwNTIzMTU0MzA5LmpwZw?x-oss-process=image/format,png#pic_center)

## 算法思路

定义数组 $pre[i]$ 为$[0…i]$里所有元素的和：那么 $pre[i] = pre[i-1]+nums[i]$ 。那么要计算任意子区间$[i,j]$的和，通过$pre[j]-pre[i]$即可得到。

```c++
int size = nums.size();//数组长度
vector<int> pre = vector(size,0);//记录[0,i]每个位置的和
//构造前缀和
for(int i = 1;i< nums.size();i++)
{
    pre[i] = pre[i-1]+nums[i];
}
//穷举所有子数组
int countK = 0;
for(int i = 0;i< nums.size();i++)//穷举所有以i开头的子串
{
    for(int j = i;j < nums.size();j++)//穷举所有子以j结尾的子串
    {
        if( pre[j] - pre[i] == k)
            countK++;
    }
}
```

## 应用实例

### 和为$k$的子数组

> 给定一个整数数组和一个整数 $k$，你需要找到该数组中和为 $k$ 的连续的子数组的个数。
>
> 示例 1 :
>
> 输入:nums = [1,1,1], k = 2
> 输出: 2 , [1,1] 与 [1,1] 为两种不同的情况。
>
> 来源：力扣（[LeetCode](https://so.csdn.net/so/search?q=LeetCode&spm=1001.2101.3001.7020)）第560道题

### 解题思路

①．暴力穷举

 可以对所有的子数组进行穷举，然后分别求和，和等于$k$则计数加$1$。

```c++
//穷举所有子串[i,j]
int countK = 0;
for(int i = 0;i< nums.size();i++)//穷举所有以i开头的子串
{
    for(int j = i;j < nums.size();j++)//穷举所有子以j结尾的子串
    {
        int sum = 0;
        for(int k = i;k <= j;k++)//对子串求和
        {
            sum += nums[k];
        }
        if( sum == k)//判断和是否为k
            countk++;
    }
}
```

 枚举所有的子串开头 i 和结尾 j，然后确定起始位置后对子串进行从头到尾求和，时间复杂度为O(n3)。

②．去除重复计算

 上述方法中对每个子串进行求和时，都是从头至尾进行累加计算，存在在大量的重复计算。因为在第二层的循环结束时已经得到了$[i,j]$的和，下次循环求$[i,j+1]$时则没必要再次从头到尾计算，直接上次的和加上$nums[j+1]$即可，时间复杂度为$O(n^2)$。

```c++
//穷举所有子串[i,j]
int countK = 0;
for(int i = 0;i< nums.size();i++)//穷举所有以i开头的子串
{
    int sum = 0;
    for(int j = i;j < nums.size();j++)//穷举所有子以j结尾的子串
    {
        sum += nums[j];//上次和的基础上加当前值即可
        if( sum == k)//判断和是否为k
            countk++;
    }
}
```

③．使用前缀和

上述方法中算$[i,j+1]$的和时，直接用$[i,j]$的和加上$nums[j+1]$，减少了子串再次从头到尾的计算，根据这一思路，我们定义数组 $pre[i]$ 为$[0…i]$里所有元素的和：那么 $pre[i] = pre[i-1]+nums[i]$ 。那么要计算任意子区间$[i,j]$的和，通过$pre[j]-pre[i]$即可得到。
```c++
int size = nums.size();//数组长度
vector<int> pre = vector(size,0);//记录[0,i]每个位置的和
//构造前缀和
for(int i = 1;i< nums.size();i++)
{
    pre[i] = pre[i-1]+nums[i];
}
//穷举所有子数组
int countK = 0;
for(int i = 0;i< nums.size();i++)//穷举所有以i开头的子串
{
    for(int j = i;j < nums.size();j++)//穷举所有子以j结尾的子串
    {
        if( pre[j] - pre[i] == k)
            countK++;
    }
}
```

pre数组作为对数据的预处理只执行一次即可，当我们需要返回任意子数组$[i,j]$的和时，可以通过 $pre[j] - pre[i]$ $O(1)$ 用的时间得到。

 但对本题来说，依然需要对数组进行两层遍历，时间复杂度为$O(n^2)$。

④．哈希表优化

 在上述方法中，第二次for循环的作用是：计算有几个 $j$ 能够使 $pre[j]$ 和 $pre[i]$ 的差等于$k$。但实际上我们不关心前缀和对应哪几项，只需要知道有几个前缀和满足条件即可。因此我们可以用哈希表记录前缀和及该和出现的次数，就可以用O(1)的时间做出判断。

 哈希表中键值对，键：从$0$到当前项的总和、值：这个前缀和出现了几次，初始化前缀和为$0$出现了$1$次。

```c++
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        unordered_map<int,int> mp;//记录 前缀和-次数
        mp[0] = 1;//初始化0出现了一次
        int count = 0,pre = 0;//仅需要记录前一个值即可，不需要用数组记录所有的前缀和
        for(int i =0;i < nums.size();i++)
        {
            pre+=nums[i];//计算前缀和
            if(mp.find(pre-k) != mp.end() )//前面结果中存在满足条件的前缀和
            {
                count+=mp[pre-k];
            }
            mp[pre]++;//相同的前缀和次数加1
        }
        return count;
    }
};
```

### 统计「优美子数组」

> 给你一个整数数组 nums 和一个整数 k。
>
> 如果某个 连续 子数组中恰好有 k 个奇数数字，我们就认为这个子数组是「优美子数组」。
>
> 请返回这个数组中「优美子数组」的数目。
>
> 示例 1：
>
> 输入：nums = [1,1,2,1,1], k = 3
> 输出：2
> 解释：包含 3 个奇数的子数组是 [1,1,2,1] 和 [1,2,1,1] 。
>
> 来源：力扣（LeetCode）第1248道题

①．题目分析

 同求连续子区间的和，只不过改为了连续子区间的奇数数字个数，同样可以使用前缀和的思路求解。

②．题解算法

```c++
class Solution {
public:
    int numberOfSubarrays(vector<int>& nums, int k) {
        unordered_map<int,int> mp;//记录 奇数个数-次数
        mp[0] = 1;
        int count = 0,pre = 0;//仅需要记录前一个值即可，不需要用数组记录所有的前缀和
        for(int i = 0;i< nums.size();i++)
        {
            if( nums[i] % 2 == 1)
                pre++;//计算前缀和
            if(mp.find(pre-k) != mp.end())//前面结果中存在满足条件的前缀和
            {
                count+=mp[pre-k];
            }
            mp[pre]++;
        }
        return count;
    }
};
```

### 连续子数组和

> 给定一个包含非负数的数组和一个目标整数 k，编写一个函数来判断该数组是否含有连续的子数组，其大小至少为 2，总和为 k 的倍数，即总和为 n*k，其中 n 也是一个整数。
>
> 示例 1:
>
> 输入: [23,2,4,6,7], k = 6
> 输出: True
> 解释: [2,4] 是一个大小为 2 的子数组，并且和为 6。
>
> 来源：力扣（LeetCode）第523道题

①．题目分析

 求连续子数组的和可以采用前缀和的思路。区间$[i,j]$的和为$k$的倍数，即求 $nums[j] - nums[i]$ 差为$k$的倍数，只要 $nums[j]$ 和 $nums[i]$ 除以$k$的余数相等即满足需求，因此只要记录每个位置前缀和除以$k$的余数即可。

 本题只需要判断是否存在，对于每个余数只需要记录第一次出现位置即可，初始化前缀和为$0$的位置下标为$-1$。

 当$k = 0$时，只要存在两个前缀和相等，则区间和为$0$，同样满足要求。

②．题解算法
```c++
class Solution {
public:
    bool checkSubarraySum(vector<int>& nums, int k) {
        unordered_map<int,int> pre_map;//记录 前缀和%k - 首次位置
        int sum = 0;
        pre_map[0]=-1;//初始化前缀和为0的下标为-1
        for(int i = 0;i < nums.size();i++)
        {
            sum += nums[i];//前缀和
            if( k != 0)
            {
                sum = sum%k;//对k求余数
            }
            if( pre_map.find(sum) != pre_map.end())//存在满足条件的前缀和
            {
                if( i - pre_map[sum] > 1)//判断长度大于1
                    return true;
            }
            else
            {
                pre_map.insert({sum,i});
            }
        }
        return false;     
    }
};
```

### 每个元音包含偶数次的最长子[字符串](https://so.csdn.net/so/search?q=字符串&spm=1001.2101.3001.7020)

> 给你一个字符串 s ，请你返回满足以下条件的最长子字符串的长度：每个元音字母，即 ‘a’，‘e’，‘i’，‘o’，‘u’ ，在子字符串中都恰好出现了偶数次。
>
> 示例 1：
>
> 输入：s = “eleetminicoworoep”
> 输出：13
> 解释：最长子字符串是 “leetminicowor” ，它包含 e，i，o 各 2 个，以及 0 个 a，u 。
>
> 来源：力扣（LeetCode）第1371道题

①．题目分析

在题$1248$统计「优美子数组」中，统计连续子区间奇数数字出现的次数，本题改为了多个字符出现的次数，同样适用前缀和的思路；在题$523$连续子数组和中，条件是前缀和是$k$的整数倍，本题中条件为偶数次，即前缀和是$2$的整数倍，即两个位置对$2$求余的结果相等即可。

- 用二进制数字记录状态

我们可以对每个元音字母都维护一个前缀和的数组，实际上我们数组中保存的是每个前缀和对$2$的余数，即$0$和$1$，因此没必要真的维护$5$个数组，使用一个二进制数字即可表示当前的组合状态，二进制数字中一位代表一个元音字母的状态。比如 $00000$表示每个元音字母都没出现。本题需求转换为找两个位置的二进制状态相等。

- 用异或翻转二进制位

用二进制位表示从开始到当前位置的状态，未出现标记成$0$，出现一次标记为$1$，再次出现则翻转为$0$，即某一个位对$1$进行异或的结果。

- 哈希表优化

本题中需求为找两个位置的状态相同，并且距离最远。只需要记录每个位置首次出现的位置即可，不断的找相同的状态，越靠后出现的离的越远。

②．题解算法

```c++
class Solution {
public:
    int findTheLongestSubstring(string s) {
        int res = 0;//保存返回结果
        s = '#'+s;//前置添加一个固定字符，初始化-1的位置
        unordered_map<int,int> mp;//每种状态首次出现的位置
        string str_temp = "aeiou";//同二进制数字中位的位置
        int pre = 0;//前缀状态，初始化为00000
        for(int i = 0;i < s.length();i++)
        {
            for(int j = 0;j< 5;j++)
            {
                if( s[i] == str_temp[j])
                    pre = pre^(1 << (4 - j))  ;//对相应位置的状态和1进行异或，0变0、0变1
            }
            if( mp.find(pre) != mp.end())//状态相等
            {
                res = max(res,i-mp[pre]);//更新最远距离
            }
            else
            {
                mp[pre] = i;//记录首次出现的位置
            }
        }
        return res;
    }
};
```

### 二维区域和检索-矩阵不可变

> 给定一个二维矩阵，计算其子矩形范围内元素的总和，该子矩阵的左上角为 (row1, col1) ，右下角为 (row2, col2)。
>
> 上图子矩阵左上角 (row1, col1) = (2, 1) ，右下角(row2, col2) = (4, 3)，该子矩形内元素的总和为 8。
>
> 示例:
>
> 给定 matrix = [
> [3, 0, 1, 4, 2],
> [5, 6, 3, 2, 1],
> [1, 2, 0, 1, 5],
> [4, 1, 0, 1, 7],
> [1, 0, 3, 0, 5]
> ]
>
> sumRegion(2, 1, 4, 3) -> 8
> sumRegion(1, 1, 2, 2) -> 11
> sumRegion(1, 2, 2, 4) -> 12
> 说明:
>
> 你可以假设矩阵不可变。
> 会多次调用 sumRegion 方法。
> 你可以假设 row1 ≤ row2 且 col1 ≤ col2。
>
> 来源：力扣（LeetCode）第304题

①．题目分析

 先构造前缀和，然后可以$O(1)$计算给出任意子矩阵的和。

②．题解算法

```c++
class NumMatrix {
public:
    vector<vector<int>> presum;//记录前缀和
    int rowcount =0;
    int colcount = 0; 
    NumMatrix(vector<vector<int>>& matrix) {
        rowcount = matrix.size();
        if( rowcount  == 0 ) return;
        else
            colcount = matrix[0].size();
        if(colcount == 0 ) return;

        presum = vector(rowcount,vector(colcount,0));//初始化前缀和数组
        //构造前缀和数组，presum[i][j] = nums[i][j] + presum[i-1][j] + presum[i][j-1] - presum[i-1][j-1]
        for(int i =0;i < rowcount;i++)
        {
            for(int j = 0;j < colcount;j++)
            {
                presum[i][j] = matrix[i][j];
                if( i-1 >= 0) presum[i][j] += presum[i-1][j] ;
                if( j-1 >= 0) presum[i][j] += presum[i][j-1] ;
                if( i-1>= 0 && j-1 >= 0) presum[i][j] -= presum[i-1][j-1] ;
            }
        }
    }
    
    int sumRegion(int row1, int col1, int row2, int col2) {
        if(rowcount == 0 || colcount == 0) return 0;
        //子矩阵和公式：presum[row2][col2] - presum[row1-1][col2] - presum[row2][col1-1] +presum[row1-1][col1-1]
        int res = presum[row2][col2];
        if( row1 -1 >=0) res -= presum[row1-1][col2];
        if( col1 -1 >=0) res -= presum[row2][col1-1];
        if(row1 -1 >= 0 && col1-1 >=0 )
            res += presum[row1-1][col1-1];
        return res;
    }
};

/**
 * Your NumMatrix object will be instantiated and called as such:
 * NumMatrix* obj = new NumMatrix(matrix);
 * int param_1 = obj->sumRegion(row1,col1,row2,col2);
 */
```

