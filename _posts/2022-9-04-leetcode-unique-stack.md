---
layout: post
title: LeetCode刷题单调栈学习笔记
date: 2022-09-04
author: lau
tags: [Leetcode, Blog]
comments: true
toc: false
pinned: false


---

单调栈学习笔记。

<!-- more -->

## 栈(Stack)

栈(Stack)：仅在同一端进行插入/删除的线性表。

是一种后进先出(Last in First Out)的结构，只在栈顶top端操作；可以使用数组or链表来实现。

例题：一个栈的入栈序列A B C D E，则不可能的输出序列为( )

1. EDCBA
2. DECBA
3. DCEAB
4. ABCDE

答案：C，B一定在A前面出栈。

### 单调栈

LeetCode $739$ ：每日温度

根据每日气温列表，请重新生成一个列表，对应位置的输入是你需要再等待多久温度才会升高的天数。如果之后都不会升高，请输入 $0$ 来代替。

例如，给定一个列表 temperatures = $[73, 74, 75, 71, 69, 72, 76, 73]$，你的输出应该是$ [1, 1, 4, 2, 1, 1, 0, 0]$。

**提示：**气温 列表长度的范围是 $[1, 30000]$。每个气温的值的都是 $[30, 100]$ 范围内的整数。

**审题：** 输入：一维数组；范围：$[1,30000]$；处理：找到下一个比自己大的元素信息；输出：一位数组；

**暴力解：** 两层for循环，实现非常简单，但时间复杂度$O(N^2)$，不满足；

```c++
class Solution {
public:
    vector<int> dailyTemperatures(vector<int>& temperatures) {
        int len=temperatures.size();
        stack<int> st;
        int j=0;
        vector<int> v(len);
        for(int i=0;i<len;i++)
        {
            while(!st.empty()&&temperatures[i]>temperatures[st.top()])
            {
                v[st.top()]=i-st.top();
                st.pop();
            }
            st.push(i);
        }
        return v;
    }
};
```

### 举一反三

单调栈的性质：

1. 单调栈里的元素具有单调性（单调递增，单调递减；单调非递减；单调非递增）；
2. 元素加入栈前，会在栈顶端把破坏栈单调性的元素都删除；
3. 使用单调栈可以找到元素向左遍历第一个比他小的元素，也可以找到元素向左遍历第一个比他大的元素；
4. 使用单调栈可以找到元素向右遍历第一个比他小的元素，也可以找到元素向右遍历第一个比他大的元素；

举例：单调递增栈->找比自己小的元素：

1. 当单调栈中的元素是单调递增的时候，根据上面我们从数组的角度阐释单调栈的性质的叙述，可以得出：

(1). 当a > b时，则将元素a插入栈顶，新的栈顶则为a；

(2). 当a < b时，则将从当前栈顶位置向前查找（边查找，栈顶元素边出栈），直到找到第一个比a小的数，停止查找，将元素a插入栈顶（在当前找到的数之后，即此时元素a找到了自己的“位置”）；同时，对于此时弹出的元素来说，其右边所对应的最小元素就为a！

### 习题LeetCode 84（柱状图中的最大矩形）

### 习题LeetCode 42（接雨水）

### 习题LeetCode 962（最大宽度坡）和习题LeetCode 402（移掉K位数字）

### Tips：刷题心得

#### 个人经验

1. 观察题目数据范围，估算时间复杂度，选择合适算法避免超时

- 当$N=10^8$时，运行时间为$1000$ms，可以根据这个基准估算；
- 当$N$数量级为$\sim 10000$,可以采用$O(N^2)$算法不会超时；
- 若$N$数量级为$10000\sim 1000000$，可以采用$O(N\log N)$算法不会超时；
- 若$N$数量级为$10^6\sim 10^8$，只能采用$O(N)/O(\log N)$算法

2. 提交前将调试打印去掉，打印很可能导致超时；
3. 需要辅助数组的话，可以在全局空间估算一下数组大小开一个大数组，同时记得初始化数组

- LeetCode对内存没有严格限制(<4G内存)
- 开数组建议在全局空间开再初始化，直接在栈上开数组可能会超栈内存

4. 描述区间相关代码，建议用左闭右开区间描述$[begin,end)$，代码会很优雅
5. LeetCode一道题代码量大概为$50$行左右，若思路清晰，时间完全充足
6. 我个人练习的时候，给自己定的约束：简单题不能超过10分钟，中等题不能超过20分钟，困难题不能超过40分钟，超过了看看他人思路、代码，完全理解后不看代码自己写出来完全提交通过，提供训练效率

#### 常用接口C/C++

C常用接口：

- malloc/free
- qsort/bsearch
- memcpy_s/memset_s
- strcpy_s/strcmp/sprintf_s/atoi
- scanf_s/printf

C++常用接口：

- std::max<T> /std::min<T>/std::swap<T>
- std::sort/std::lower_bound/std::upper_bound/std::unique
- std::vector<T>/std::map<T>

#### 常用算法C/C++：内存分配

分配二级指针，首先分配第一维，然后根据第一维分配第二维。

```c++
// C部分
int n = 20, m = 30;
pArray = malloc(n * sizeof(int *)); // 第一维是int *
for (int i = 0; i < n; ++i) {
    pArray[i] = malloc(m * sizeof(int)); // 注意第二维是int
}
// C++部分
int n = 20, m = 30;
vector<vector<int>> arr(n, vector<int>(m, 0));// 全部初始化为0

// 同理可得三维指针分配方式：
int ***pArr = NULL;
int n = 20, m = 30, k = 40;
pArr = malloc(n * sizeof(int **));
for (int i = 0; i < n; ++i) {
    pArr[i] = malloc(m * sizeof(int *));
    for (int j = 0; j < m; ++j) {
        pArr[i][j] = malloc(k * sizeof(int));
    }
}

// C++
int n = 20, m = 30, k = 40;
vector<vector<vector<int>>> arr(n, vector<vector<int>>(m, vector<int>(k, 0)));
```

#### 常用算法C/C++：去重

去重在很多题目中应该比较重要，相当基础，这里给出去重的模板，应该熟悉掌握，去重需要保证传入的数组长度>0，返回值为去重后的数组长度。

- 若数组已经排序过，则去重相同元素；
- 若未排序，则去重相邻相同元素；

```c
int unique(int *array, int arraySize) {
    int i = 0;
    for (int j = 1; j < arraySize; ++j) {
        if (array[i] != array[j]) {
            array[++i] = array[j];
        }
    }
    return  i + 1;
}

// C++ STL  unique/erase 
nums.erase(unique(nums.begin(), nums.end()), nums.end());
```

LeetCode习题：(移除有序数组重复元素)(https://*leetcode*-cn.com/problems/*remove*-*duplicates*-from-*sorted*-array)

*Leetcode*每日一题:83.*remove*-*duplicates*-from-*sorted*-list(删除排序链表中的*重复*元素)
