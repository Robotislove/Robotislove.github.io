---
layout: post
title: LeetCode刷题滑动窗口学习笔记
date: 2022-08-28
author: lau
tags: [Leetcode, Blog]
comments: true
toc: false
pinned: false

---
滑动窗口学习笔记。

<!-- more -->

## 滑窗的使用
算法技巧的思路非常简单，就是维护一个窗口，不断滑动，然后更新答案。这个算法技巧的时间复杂度是O(N)，比字符串暴力算法要高效得多。一般用一个map或者set来维护窗口内元素的特征，比如去重复。

### 滑动窗口的定式

- 声明两个下标index：left、right
- 外层给一个while循环，right遍历数组或字符串长度
- 增大窗口，解题逻辑
- 缩小窗口，解题逻辑，left++
- right++

```c++
/* 滑动窗口算法框架 */
void slidingWindow(string s, string t) {
    unordered_map<char, int> need, window;
    for (char c : t) need[c]++;

    int left = 0, right = 0;
    int valid = 0; 
    while (right < s.size()) {
        // c 是将移入窗口的字符
        char c = s[right];
        // 右移窗口
        right++;
        // 进行窗口内数据的一系列更新
        ...

        /*** debug 输出的位置 ***/
        printf("window: [%d, %d)\n", left, right);
        /********************/

        // 判断左侧窗口是否要收缩
        while (window needs shrink) {
            // d 是将移出窗口的字符
            char d = s[left];
            // 左移窗口
            left++;
            // 进行窗口内数据的一系列更新
            ...
        }
    }
}
```



```c++
int left = 0, right = 0;
while( right < s.size() ) {
  // 增大窗口
  window.add(s[right]);
  while(window needs shrink) {
    // 缩小窗口
    window.remove(s[left]);
    left++;
  }
  right++;
}
```
例题：3. [无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/); 395. [至少有K 个重复字符的最长子串](https://leetcode.cn/problems/longest-substring-with-at-least-k-repeating-characters/); 424. [替换后的最长重复字符](https://leetcode.cn/problems/longest-repeating-character-replacement/); 239. [滑动窗口最大值](https://leetcode.cn/problems/hua-dong-chuang-kou-de-zui-da-zhi-lcof/)。

**编码注意事项：** right++、left++的位置
- right++一般喜欢扔在外层while的末尾；
- 注意缩小窗口的时候是否会用到right值，再确定right在哪里++；
- left++固定在缩小窗口的逻辑里面；
- 如果有记录元素的位置，left直接跳到对应元素的位置；

- 窗口初始化
  设置窗口初始化值为 0 ，即 left 和 right 都位于序列的左端位置。
- 窗口扩张
  窗口扩张时 left 指针保持不动，向右移动 right 指针；在窗口扩张的初期，窗口右侧纳入新的元素对结果的影响是积极的，有以下两种情况：
  ①．新加入的元素使窗口得到的结果更优，则每一步都是一个当前最优解；停止扩张的临界条件是新加入的元素刚好使窗口不符合条件。
  ②．新加入的元素使窗口离达到目标条件更近；停止扩展的临界条件是新加入的元素刚好使窗口符合目标条件。
  当窗口达到停止扩张的临界条件时，继续扩张对结果的影响是消极的，此时应该进行窗口收缩。
- 窗口收缩
  窗口收缩时 right 指针保持不动，向右移动 left 指针；窗口收缩的初期，窗口左侧丢弃的元素对结果的影响的积极的，有以下两种情况：
  ①．若窗口停止扩张的临界条件是新加入的元素刚好使窗口不符合目标条件，则丢弃一个旧元素可能使窗口满足条件，停止收缩的临界条件是丢弃一个元素使窗口刚好满足目标条件。
  ②．若窗口停止扩张的临界条件是新加入的元素刚好使窗口符合目标条件，则丢弃一个旧元素可能使窗口的结果更优，停止收缩的临界条件是丢弃一个元素使窗口刚好不满足目标条件。
  当窗口达到停止收缩的临界条件时，继续收缩对结果的影响是消极的，此时应该进行窗口扩张。
- 滑动停止
  滑动窗口的过程就是不断的进行窗口扩张与收缩，当窗口扩张对结果的影响是积极时就进行窗口扩张，当窗口收缩对结果的影响时积极时就进行窗口收缩，不断重复这个过程。当right 指针到达序列末尾时，窗口滑动停止。

### 应用实例

[剑指 Offer 59 - I. 滑动窗口的最大值](https://leetcode.cn/problems/hua-dong-chuang-kou-de-zui-da-zhi-lcof/)

我们都知道,若是一个数字$A$进入窗口后,若是比窗口内其他数字都大,那么这个数字之 前的数字都没用了,因为它们必定会比$A$早离开窗口,在$A$离开之前都争不过$A$,所以$A$在进入时要依次从后排除掉前面的小值。而因为窗口符合先进先出的原理,因此可以考虑双向队列。

- step 1 :维护一个双向队列,用来存储数列的下标。

- step 2 :首先检查窗口大小与数组大小。

- step 3: 先遍历第一个窗口,如果即将进入队列的下标的值大于队列后方的值,依次将小于的值拿出来去掉,再加入,保证队列是递增序。

- step 4: 遍历后续窗口,每次取出队首就是最大值,如果某个下标已经过了窗口,  则从队列前方将其弹出。

- step 5: 对于之后的窗口,重复step 3 ,直到数组结束。

```c++
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        if(!nums.size()) return vector<int> {};
        int n = nums.size();
        priority_queue<pair<int, int>> q;   // 优先队列，构建当前元素及其在数组中坐标的对应关系
        for (int i = 0; i < k; ++i) {   // 第一个滑动窗口的值和位置的对应关系
            q.emplace(nums[i], i);
        }
        vector<int> ans = {q.top().first};  // 初始化将第一个滑动窗口中元素最大的值存到vector中
        for (int i = k; i < n; ++i) {   // 滑动窗口右移
            q.emplace(nums[i], i);
            while (q.top().second <= i - k) {   // 这个值在数组nums中的位置出现在滑动窗口左边界的左侧
                q.pop();    // 永久移除该值，直到找到一个在滑动窗口中的最大值
            }
            ans.push_back(q.top().first);   // 将该最大值压入vector中
        }
        return ans;
    }
};
```

```c++
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        if(!nums.size()) return vector<int> {}; // 判空
        int n = nums.size();
        deque<int> q;   // 双向队列
        // 将第一个滑动窗口中的单调队列对应的坐标构建出来
        // q 中存单调队列中对应的nums坐标
        // 若新元素比 q 结尾坐标的元素值大
        // 从 q 结尾开始移除坐标 （始终保持最大值在头）
        // 否则将新元素的坐标进入队列
        for(int i = 0; i < k; ++i){
            while(!q.empty() && nums[i] > nums[q.back()]){  
                q.pop_back();
            }
            q.push_back(i);
        }
        vector<int> ans = {nums[q.front()]};    // 存滑动窗口的最大值
        // 滑动窗口右移，继续将新元素与 q 结尾的坐标比较
        for(int i = k; i < n; ++i){
            while(!q.empty() && nums[i] > nums[q.back()]){
                q.pop_back();
            }
            q.push_back(i);
            while(q.front() <= i - k)   // 若 q 头坐标在滑动窗口左边界外侧
                q.pop_front();  // 移除 q 头坐标
            ans.push_back(nums[q.front()]); // q 的头坐标的值即滑动窗口的最大值压入vector
        }
        return ans;
    }
};
```

