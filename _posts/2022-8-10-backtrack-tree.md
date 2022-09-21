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
> leetcode [面试题 08.04. 幂集](https://leetcode.cn/problems/power-set-lcci/)
> 给你一个整数数组 nums ，其中可能包含重复元素，请你返回该数组所有可能的子集（幂集）。
> 解集 不能 包含重复的子集。返回的解集中，子集可以按 任意顺序 排列。



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

> [77. 组合](https://leetcode.cn/problems/combinations/)
>
> 给定两个整数 `n` 和 `k`，返回范围 `[1, n]` 中所有可能的 `k` 个数的组合。
>
> 你可以按 **任何顺序** 返回答案。

```c++
class Solution {
public:
    vector<vector<int>> combine(int n, int k) {
        vector<int> tmp;
        vector<vector<int>> res;
        dfs(1, res, tmp, k, n);
        return res;
    }

    void dfs(int sign, vector<vector<int>>& res, vector<int> tmp, int k ,int n) {
        
		if (tmp.size() == k) {
            res.push_back(tmp); 
            return;
        }
        for (int i = sign; i <= n; i++) { // 1 < 4
            tmp.push_back(i); // tmp:1 2
            dfs(i+1, res, tmp, k, n);
            tmp.pop_back();
        }
    }
};
```

思路：用这个题来整理一下回溯问题的模板。题目的意思其实就是找组合，组合的元素个数是 K。组合无序，**所以使用过的数字不能再使用，否则就重复了**，所以越往后找其实需要遍历的越少。

递归函数参数：需要三个，分别是**起始位置、要求的区间末尾数字、组合的大小**。凡是组合类的问题，都需要一个起始位置作为参数，因为下一个递归需要从下一个位置开始。递归函数一般不需要返回值，但也有例外情况，有的情况加上一个返回值会提高搜索效率。

递归的逻辑：把当前的数字添加到结果中，然后在递归下一个数字。当递归返回时再弹出。

递归终止条件：只要当前的结果大小等于 K ，那这就是一个符合条件的结果，添加到结果集中。

> No 40. 组合总和 II(中等)
> 来源：力扣（LeetCode）
> 链接：https://leetcode-cn.com/problems/combination-sum-ii/
>
> 题目描述：
>
> 给定一个候选人编号的集合 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。candidates 中的每个数字在每个组合中只能使用 一次 。注意：解集不能包含重复的组合。
> 

**思路**：这个题有一个特殊情况，那就是给定的数组中可能会有重复的元素，那么如果这个元素使用过了，后面再使用，那就会导致重复，但是我们又不能在一开始就删除掉重复的元素，否则就和给定的数组不一样了，结果肯定会不正确。所以我们只能在遍历的时候，一边遍历，一边进行去重的操作。

我的去重思路是这样：我们按照正常的递归回溯先操作，当递归返回的时候，将要进行下一次 FOR 循环 的时候，进行去重，如果下一个数字和这个使用过的数字是一样的，那就跳过，注意要用while循环跳过，因为可能不止一个重复的。要想进行这样的操作必须首先对原数组进行排序，让相同的元素挨在一起。可是排序对原数组进行了修改，不会导致其它问题吗？这里我们找的是组合，和顺序没有关系，只要是这些数字，找到的就是这些组合，不会因为数组的顺序变化而导致组合的变化。
```c++
class Solution {
public:
    //标准递归回溯
    vector<vector<int>> res;
    vector<int> res_temp;
    void backTracking(vector<int>& nums, int target, int sum, int startIndex) {
        //符合条件
        if(sum == target) {
            res.push_back(res_temp);
            return;
        }
        //不符合条件
        if(sum > target) {
            return;
        }
        //递归
        for(int i = startIndex; i < nums.size(); i++) {
            sum += nums[i];
            res_temp.push_back(nums[i]);
            backTracking(nums, target, sum, i + 1);
            res_temp.pop_back();
            sum -= nums[i];
            //关键步骤：当操作结束递归返回，准备开始下一轮时，如果发现当前数字和后面数字相同，那就跳过，否则会重复
            //要用while，因为可能不止一个相同，但是这个要求nums有序; 
            while(i < nums.size() - 1 && nums[i] == nums[i + 1]) {
                i++; 
            }
        }
    }
    vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
        //这里排序是为了后面排除重复集合，让相同的元素放在一起
        sort(candidates.begin(), candidates.end());
        backTracking(candidates, target, 0, 0);
        return res;
    }
};
```

> **No** 47. 全排列 II**(中等）**
>
> > 来源：力扣（LeetCode）
> > 链接：​[https://leetcode-cn.com/problems/permutations-ii/](https://leetcode-cn.com/problems/swap-nodes-in-pairs/)
> >
> > **题目描述：**
> >
> > 给定一个可包含重复数字的序列 `nums` ，***按任意顺序*** 返回所有不重复的全排列。

```c++
示例 1：
输入：nums = [1,1,2]
输出：
[[1,1,2],
 [1,2,1],
 [2,1,1]]
 
示例 2：
输入：nums = [1,2,3]
输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

```c++
class Solution {
public:
    //优化：也可以对nums先排序，因为是求全排列，所以顺序无所谓，之后利用while循环来代替记录同层重复的容器即可
    vector<vector<int>> res;
    vector<int> res_temp;
    void backTracking(vector<int>& nums, vector<int>& usedNum) {
        if(res_temp.size() == nums.size()) {
            res.push_back(res_temp);
            return;
        }
        //usedNum用来控之前用过的数字不能重复使用，递归中用，深度
        for(int i = 0; i < nums.size(); i++) {
            if(usedNum[i] == 0){
                //用过的数字置1
                usedNum[i] = 1;
                res_temp.push_back(nums[i]);
                backTracking(nums, usedNum);
                //递归返回时弹出并置0
                res_temp.pop_back();
                usedNum[i] = 0;
            }
            // 当前数字使用过且下一个数字和当前数字相同
            while(i < nums.size() - 1 && nums[i] == nums[i + 1] && usedNum[i] == 1){
                i++;
            }
        }
    }
 
    //本题维护了两个容器，来记录同层和递归的时候使用过的数字
    // vector<vector<int>> res;
    // vector<int> res_temp;
    // void backTracking(vector<int>& nums, vector<int>& usedNum) {
    //     if(res_temp.size() == nums.size()) {
    //         res.push_back(res_temp);
    //         return;
    //     }
    //     //numSet用来控制同层不能重复使用，for循环中的，层间
    //     //usedNum用来控之前用过的数字不能重复使用，递归中用，深度
    //     unordered_set<int> numSet;
    //     for(int i = 0; i < nums.size(); i++) {
    //         if(numSet.find(nums[i]) == numSet.end() && usedNum[i] == 0){
    //             //同层使用过的数字
    //             numSet.insert(nums[i]);
    //             //用过的数字置1
    //             usedNum[i] = 1;
    //             res_temp.push_back(nums[i]);
    //             backTracking(nums, usedNum);
    //             //递归返回时弹出并置0
    //             res_temp.pop_back();
    //             usedNum[i] = 0;
    //         }
    //     }
    // }
    vector<vector<int>> permuteUnique(vector<int>& nums) {
        vector<int> usedNum(nums.size(), 0);
        sort(nums.begin(), nums.end());
        backTracking(nums, usedNum);
        return res;
    }
};
```

```c++
class Solution {
    vector<int> vis;

public:
    void backtrack(vector<int>& nums, vector<vector<int>>& ans, int idx, vector<int>& perm) {
        if (idx == nums.size()) {
            ans.emplace_back(perm);
            return;
        }
        for (int i = 0; i < (int)nums.size(); ++i) {
            if (vis[i] || (i > 0 && nums[i] == nums[i - 1] && !vis[i - 1])) {
                continue;
            }
            perm.emplace_back(nums[i]);
            vis[i] = 1;
            backtrack(nums, ans, idx + 1, perm);
            vis[i] = 0;
            perm.pop_back();
        }
    }

    vector<vector<int>> permuteUnique(vector<int>& nums) {
        vector<vector<int>> ans;
        vector<int> perm;
        vis.resize(nums.size());
        sort(nums.begin(), nums.end());
        backtrack(nums, ans, 0, perm);
        return ans;
    }
};

```



> No 17. 电话号码的字母组合(中等）
> 来源：力扣（LeetCode）
> 链接：https://leetcode-cn.com/problems/n-ary-tree-level-order-traversal/
>
> 题目描述：
>
> 给定一个仅包含数字 $2-9$ 的字符串，返回所有它能表示的字母组合。答案可以按 任意顺序 返回。给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。
> ![](https://img-blog.csdnimg.cn/img_convert/c7b7d59e049a3996f8c0c832c102f073.png)

```c++
示例 1：
输入：digits = "23"
输出：["ad","ae","af","bd","be","bf","cd","ce","cf"]
 
示例 2：
输入：digits = ""
输出：[]
 
示例 3：
输入：digits = "2"
输出：["a","b","c"]
```

思路：这一题刚拿到肯定会有点懵，感觉好像知道要怎么做，但是具体的细节又不是很清楚，其实这是因为对回溯还没有完全理解。拿到这样的题，首先第一步要建立映射关系，也就是**把按键和字母对应起来**。这道题目肯定是建立一个**字符串数组**了，因为每一个按键上有多个字母，到时候肯定要用到其中一个或几个，需要遍历。 

其次要考虑一下，如何对输入的数字进行拆分。例如输入了$23$，我们要分别对$2$和$3$对应的字符串进行梳理，如果输入$234$，那就分别对$2$，$3$和$4$对应的字符串进行处理。其实就是在 $N$ 个数组中找所有的组合。之前做过在一个数组中找组合的，现在变成了 $N$ 个。

关键点：用一个 $index $表示现在用到的是输入数字的第几个数字。例如输入$ 23 $，那么$ index = 0$ 表示现在使用的是$2$，$index = 1$表示现在使用的是$3$。而用这个 $index $可以拿到这个数字对应的数组。拿到这个数组就可以进行递归遍历了。

```c++
class Solution {
public:
    const string map[10] = { {""},{""},{"abc"},{"def"},{"ghi"},{"jkl"},{"mno"},{"pqrs"},{"tuv"},{"wxyz"} };
    vector<string> res;
    string str;
    // 关键在于用一个index就可以实现对digits的数字逐个获取同时实现获取不同数字对应的字母数组
    void backTracking(string digits, int index) {
        if(index == digits.size()) {
            res.push_back(str);
            return;
        }
        // 拿到digits的一个数字，index其实可以表示这是第几层，也就是深度
        int numOfDigits = digits[index] - '0';
        // 拿到这个数字代表的字母
        string lettersOfNum = map[numOfDigits];
        // 不同的index对应不同的字母集，并且能够在递归的时候切换
        for(int i = 0; i < lettersOfNum.size(); i++) {
            str.push_back(lettersOfNum[i]);
            // 这里的index + 1 自带回溯，当递归结束的时候index还是index，也可以在递归前后+1-1
            backTracking(digits, index + 1);
            str.pop_back();
        }
    }
    vector<string> letterCombinations(string digits) {
        if(digits.size() == 0){
            return res;
        }
        backTracking(digits, 0);
        return res;
    }
};
```



> No 131. 分割回文串(中等）
> 来源：力扣（LeetCode）
> 链接：https://leetcode-cn.com/problems/palindrome-partitioning/
>
> 题目描述：
>
> 给你一个字符串$ s$，请你将 $s$ 分割成一些子串，使每个子串都是 回文串 。返回 $s$ 所有可能的分割方案。回文串 是正着读和反着读都一样的字符串。
>

```c++
示例 1：
输入：s = "aab"
输出：[["a","a","b"],["aa","b"]]
 
示例 2：
输入：s = "a"
输出：[["a"]]
```

```c++
class Solution {
public:
    vector<vector<string>> res;
    vector<string> res_temp;
    //判断是不是回文串
    bool judgeStr(const string& s, int leftIndex, int rightIndex){
        for(int i = leftIndex, j = rightIndex; i < j; i++, j--){
            if(s[i] != s[j]){
                return false;
            }
        }
        return true;
    }
    void backTracking(const string& s, int stratIndex) {
        //如果分割完以后发现都是回文串，那就把这个分割方案写到分割方案集合中
        if(stratIndex >= s.size()){
            res.push_back(res_temp);
            return;
        }
        //判断从startIndex 到 i 的字符串是不是回文串，startIndex = 0的这一层，子串的长度依次是1,2,3...s.size()
        for(int i = stratIndex; i < s.size(); i++) {
            //如果不是回文串那就再添加一个字符进来看看是不是回文串，所以直接下一轮循环添加一个字符
            //当前子串不是回文串就不用递归了，因为题目要求所有子串都是回文串
            if(judgeStr(s, stratIndex, i ) == false){ 
                continue;
            }
            //如果是回文串，就把当前这个字符串，写到分割方案中
            if(judgeStr(s, stratIndex, i ) == true) {
                string str = s.substr(stratIndex, i - stratIndex + 1);
                res_temp.push_back(str);
            }
            //并递归下一个字符，从下一个字符开始，看看有没有回文串，也就是深度+1
            //这层也是像第一层一样，子串长度是1,2,3,...s.size()，只不过这里的s是去掉第一个字符的s
 
            backTracking(s, i + 1);
            //递归返回要把之前记录进来的弹出
            res_temp.pop_back();
        }
    }
    vector<vector<string>> partition(string s) {
        backTracking(s, 0);
        return res;
    }
};
```



> No 491. 递增子序列(中等）
> 来源：力扣（LeetCode）
> 链接：https://leetcode-cn.com/problems/increasing-subsequences/
>
> 题目描述：
>
> 给你一个整数数组 nums ，找出并返回所有该数组中不同的递增子序列，递增子序列中 至少有两个元素 。你可以按 任意顺序 返回答案。数组中可能含有重复元素，如出现两个整数相等，也可以视作递增序列的一种特殊情况。
> 

```c++
示例 1：
输入：nums = [4,6,7,7]
输出：[[4,6],[4,6,7],[4,6,7,7],[4,7],[4,7,7],[6,7],[6,7,7],[7,7]]
 
示例 2：
输入：nums = [4,4,3,2,1]
输出：[[4,4]]
```

**思路：**这一题其实也是换汤不换药，但是这是一个求组合的题目，需要起始位置。不同点是加入的条件判断变成了只有比当前保存的结果数组最后一个数字大的才能加入这个结果数组，这样可以保证这个结果数组是**升序**。但是这个题目给定的数组是有重复的数字的，如果重复使用就会导致出现重复的序列。而且我们不能对这个原数组进行排序，然后按照之前总结过的当时去重，因为这个题目涉及到求子序列，改变原数组的顺序会改变子序列的顺序。那么我们只能用之前总结过的 树层 去重，在递归的 for 循环之前定义一个数组标记 当前树层 使用过的数字，后面在加入对是否使用过的判断条件即可。

小陷阱：在终止条件这里，每当结果数组大小大于2 的时候，就可以作为一个升序子序列了，但是这里一定不能直接返回，因为这个题目不是求最终的结果，而是相当于从根节点到叶子节点的每一个节点，只要符合条件都是一个升序子序列。那什么时候返回呢？起始位置超过数组最后一个位置的下标的时候就终止了。

```c++
class Solution {
public:
    vector<vector<int>> res;
    vector<int> res_temp;
    void backTracking(vector<int>& nums, int startIndex) {
        if(res_temp.size() >= 2) {
            res.push_back(res_temp);
        }
        if(startIndex == nums.size()){
            return;
        }
        // 使用set来对本层元素进行去重，每一层递归都定义了一个新的，所以是记录每一层用过的数字
        // 以[4,7,6,7]为例，第一层的元素是4,7,6,7;然后7重复,只使用4,7,6; 
        // 然后4,7,6又作为根节点，节点4下面有孩子7,6,7,其中7重复了,只使用一个;节点7下面有孩子6,7;节点6下面有孩子7;
        // 所以所谓层，其实就是根节点下所有的孩子,同一父节点下的同层上使用过的孩子就不能在使用了
        // 所以所谓深，其实就是根节点下到叶子节点的深度，在利用递归一边往下走一边记录节点的时候，每次记录后就把符合条件的写到临时结果集中
        // 直到所有以4开头的子集都遍历完，也就是以4为根节点的所有路径都遍历完，把这个临时结果集添加到总结果集中。同层其他节点和孩子节点也一样。
        // 这里不能使用之前的去重方法，因为之前的去重方法要求相同的数字在一起，本题不能排序，所以只能用一个不能有重复元素的unordered_set。
        unordered_set<int> deduplication; 
        // 也可以使用一个数组代替，因为题目给定了数字的范围，可以用一个大小为200的数组来记录 
        // int usedNum[201] = {0};       
        for(int i = startIndex; i < nums.size(); i++) {
            // 不仅要满足nums[i]要比res_temp末尾的值大，还要满足nums[i]在本层中没有使用过
            // 如果使用数组 usedNum[nums[i] + 100] == 0;如果使用set：deduplication.find(nums[i]) == deduplication.end()
            if((res_temp.empty() == true || nums[i] >= res_temp.back()) &&  deduplication.find(nums[i]) == deduplication.end()) 			{
                //把当前层使用过的数字记录下来，同一父节点下的同层上使用过的元素就不能在使用了
                deduplication.insert(nums[i]);
                // usedNum[nums[i] + 100] = 1;
                res_temp.push_back(nums[i]);
                backTracking(nums, i + 1);
                res_temp.pop_back();
            }
            else{
                //因为是无序数组，虽然当前数字不符合要求，但是不知道下一个是不是符合
                continue;
            }
        	}
    	}
    //要深刻理解回溯的层和深的概念
    vector<vector<int>> findSubsequences(vector<int>& nums) {
        if(nums.size() < 2){
            return res;
        }
        backTracking(nums, 0);
        return res;
    }   
};
```

> No 51.N皇后(困难）
> 来源：力扣（LeetCode）
> 链接：https://leetcode-cn.com/problems/n-queens/
>
> 题目描述：
>
> n 皇后问题 研究的是如何将 n 个皇后放置在 n×n 的棋盘上，并且使皇后彼此之间不能相互攻击。给你一个整数 n ，返回所有不同的 n 皇后问题 的解决方案。每一种解法包含一个不同的 n 皇后问题 的棋子放置方案，该方案中 'Q' 和 '.' 分别代表了皇后和空位。
>



![](https://img-blog.csdnimg.cn/img_convert/29faf73bee151a4a7cd85aee43fd6e31.png)

```c++
示例 1：
输入：n = 4
输出：[[".Q..","...Q","Q...","..Q."],["..Q.","Q...","...Q",".Q.."]]
解释：如上图所示，4 皇后问题存在两个不同的解法。
 
示例 2：
输入：n = 1
输出：[["Q"]]
```

**思路：**这一题和前面的也是类似，这一次需要判断的是当前要放置棋子的位置是否合法。这里的棋子是皇后，这里普及一点点国际象棋小知识，皇后可以横着、竖着、斜着走。所以这里的合法判断就变成了判断当前位置的同行、同列、斜线是否存在皇后。这里一定要想清楚斜着走的话，行和列是怎么加减的，而且只需要向上判断，因为下面的棋盘还没处理。

首先进行棋盘的初始化，定义一个字符串数组，并全部初始化为题目要求的 点 。然后就开始进行递归，递归的是需要期盼的大小，棋盘和当前所在的行数。因为一行肯定只能放一个皇后，所以以行为单位进行for循环。在递归之前进行条件判断即可，是合法的位置就变为皇后，然后进行递归，如果不是合法的位置，那就进行下一轮循环，看看这一行的下一个位置合不合法。递归返回时记得要回溯，撤销之前的操作。

```c++
class Solution {
public:
    vector<vector<string>> res;
    //不能同行，不能同列，不能同斜线
    bool judgePosition(int row, int col, int n, vector<string>& res_temp){
        //同行,不用处理，单层搜索的过程中，每一层递归，只会选for循环（也就是同一行）里的一个元素，下一次就去下一行了，所以不用去重了
        // for(int col_i = 0; col_i < n; col_i++){
        //     if(res_temp[row][col_i] == 'Q'){
        //         return false;
        //     }
        // }
 
        //同列
        for(int row_i = 0; row_i < n; row_i++){
            if(res_temp[row_i][col] == 'Q'){
                return false;
            }
        }  
 
        //同斜线,注意斜线有2个方向，只需要往上查找就行，因为下面的行还没有进行处理
        //45度，左上，row控制上下，col控制左右
        for(int i = row - 1, j = col - 1; i >= 0 && j >= 0; i--,j--){
            if(res_temp[i][j] == 'Q'){
                return false;
            }
        }
        //135度, 右上
        for(int i = row - 1, j = col + 1; i >= 0 && j < n; i--,j++){
            if(res_temp[i][j] == 'Q'){
                return false;
            }
        }
        return true;
    }
    void backTracking(int n, int row, vector<string>& res_temp) {
        if(row == n) {
            res.push_back(res_temp);
            return;
        }
        for(int col = 0; col < n; col++) {
            if( judgePosition(row, col, n, res_temp) == true) {
                res_temp[row][col] = 'Q';
                backTracking(n, row + 1, res_temp);
                res_temp[row][col] = '.';
            }
        }
    }
    vector<vector<string>> solveNQueens(int n) {
        vector<string> res_temp(n, string(n, '.'));
        backTracking(n, 0, res_temp);
        return res;
    }
};
```

> No 37. 解数独(困难）
> 来源：力扣（LeetCode）
> 链接：https://leetcode-cn.com/problems/sudoku-solver/
>
> 题目描述：
>
> 编写一个程序，通过填充空格来解决数独问题。数独的解法需 遵循如下规则：
>
> 数字 1-9 在每一行只能出现一次。
> 数字 1-9 在每一列只能出现一次。
> 数字 1-9 在每一个以粗实线分隔的 3x3 宫内只能出现一次。（请参考示例图）
> 数独部分空格内已填入了数字，空白格用 '.' 表示。

![](https://img-blog.csdnimg.cn/img_convert/f4fb9f4132522494a6f2eaed52290a01.png)

```c++
示例 1：
输入：board = [
["5","3",".",".","7",".",".",".","."],
["6",".",".","1","9","5",".",".","."],
[".","9","8",".",".",".",".","6","."],
["8",".",".",".","6",".",".",".","3"],
["4",".",".","8",".","3",".",".","1"],
["7",".",".",".","2",".",".",".","6"],
[".","6",".",".",".",".","2","8","."],
[".",".",".","4","1","9",".",".","5"],
[".",".",".",".","8",".",".","7","9"]]
 
输出：[
["5","3","4","6","7","8","9","1","2"],
["6","7","2","1","9","5","3","4","8"],
["1","9","8","3","4","2","5","6","7"],
["8","5","9","7","6","1","4","2","3"],
["4","2","6","8","5","3","7","9","1"],
["7","1","3","9","2","4","8","5","6"],
["9","6","1","5","3","7","2","8","4"],
["2","8","7","4","1","9","6","3","5"],
["3","4","5","2","8","6","1","7","9"]]
 
解释：输入的数独如上图所示，唯一有效的解决方案如下所示：
```

![](https://img-blog.csdnimg.cn/img_convert/a7eb44484738d96ca1893d1c9a888977.png)

**思路：**看到这个题，回想起了之前上中学被数独游戏支配的恐惧，要是中学的时候会这个那多好，哈哈。这个题其实本质上还是换汤不换药，只不过递归时要进行的条件判断复杂了一些。

合法判断：同行、同列不能有重复数字，当前这个3*3 的格子内不能有重复数字。确定每一个小格子的起始位置想了挺久，后来发现是自己想复杂了，直接用 当前行 除以 3然后在 * 3，那就是起始位置，其实是利用了 整形除法的取整特性。本质上还是映射，就是把这个格子内的行列数都映射在这个格子的起始位置。

整体思路：利用两个 for 循环，遍历每一个位置，如果这个位置是数字，那就跳过，直接遍历下一个数字，如果是空格，那就对这个位置以及要放的数字进行合法判断，如果合法那就添加一个数字，如果不合法那就换一个数字再判断。如果这9个数字都用完了还不合法，那就无解。

这里的递归函数也有返回值，因为如果找到了合法的数独解，后面的就不用再遍历了。
```c++
class Solution {
public:
    // 需要返回值但不需要终止条件
    //返回值的作用是解数独找到一个符合的条件立刻就返回，相当于找从根节点到叶子节点一条唯一路径
    //不需要终止条件是因为要遍历整个棋盘
    bool backTracking(vector<vector<char>>& board){
        for(int i = 0; i < board.size(); i++){
            for(int j = 0; j < board[i].size(); j++){
                //这个位置上有数字
                if(board[i][j] != '.'){
                    continue;
                }
                for(char c = '1'; c <= '9'; c++){
                    if(judgeValid(i, j, c, board) == true){
                        //设置数字
                        board[i][j] = c;
                        if(backTracking(board) == true){
                            return true;
                        }
                        //回溯撤销
                        board[i][j] = '.';
                    }
                }
                //这9个数字都试完了都不行
                return false;
            }
        }
        return true;
    }
 
    bool judgeValid(int row, int col, char val, vector<vector<char>>& board){
        //同行
        for(int i = 0; i < 9; i++) {
            if(board[row][i] == val){
                return  false;
            }
        }
        //同列
        for(int j = 0; j < 9; j++) {
            if(board[j][col] == val){
                return  false;
            }
        }
        //3*3网格内是否重复
        int startRow = (row / 3) * 3;
        int startCol = (col / 3) * 3;
        for(int i = startRow; i < startRow + 3; i++){
            for(int j = startCol; j < startCol + 3; j++){
                if(board[i][j] == val){
                    return false;
                }
            }
        }
        return true;
    }
    void solveSudoku(vector<vector<char>>& board) {
        backTracking(board);
    }
};
```

