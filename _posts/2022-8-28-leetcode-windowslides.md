---
layout: post
title: LeetCode刷题滑动窗口学习笔记
date: 2022-08-28
author: lau
tags: [Leetcode, Blog]
comments: true
toc: true
pinned: false

---
滑动窗口学习笔记。

<!-- more -->

## 滑窗的使用
算法技巧的思路非常简单，就是维护一个窗口，不断滑动，然后更新答案。这个算法技巧的时间复杂度是O(N)，比字符串暴力算法要高效得多。

### 滑动窗口的定式

- 声明两个下标index：left、right
- 外层给一个while循环，right遍历数组或字符串长度
- 增大窗口，解题逻辑
- 缩小窗口，解题逻辑，left++
- right++

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

