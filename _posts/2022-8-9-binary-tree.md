---
layout: post
title: leetcode刷题二叉树知识笔记
date: 2022-08-9
author: lau
tags: [Leetcode,Archive]
comments: true
toc: false
pinned: false
---
leetcode刷题二叉树知识笔记。

<!-- more -->

## 二叉树遍历和构建二叉树

```c++
#include <iostream>
#include <vector>

#include <queue>

using namespace std;

struct TreeNode {
    // int val;
    // TreeNode *left;
    // TreeNode *right;
    // TreeNode(int x) : val(x), left(NULL), right(NULL) {}
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};


TreeNode* construct_binary_tree(const vector<int>& vec){
    vector<TreeNode*> vecTree (vec.size(), NULL);
    TreeNode* root = NULL;
    // value 插入treenode节点
    for (int i = 0; i < vec.size(); i++){
        TreeNode* node = NULL;
        if (vec[i] != -1) node = new TreeNode(vec[i]);
        vecTree[i] = node;
        if (i == 0) root = node;
    }
    // treenode的left 和 right 分别指明指向
    for( int i = 0; i * 2 + 2 < vec.size(); i++){
        if (vecTree[i] != NULL){
            vecTree[i]->left = vecTree[i * 2 + 1];
            vecTree[i]->right = vecTree[i * 2 + 2];
        }
    }
    return root;
}

// 层次遍历打印

void print_binary_tree(TreeNode* root){
    queue<TreeNode*> que;
    if (root != NULL) que.push(root);
    vector<vector<int>> result;
    while (!que.empty()) {
        int size = que.size();
        vector<int> vec;
        for (int i = 0; i < size; i++){
            TreeNode* node = que.front();
            que.pop();
            if (node != NULL){
                vec.push_back(node->val);
                que.push(node->left);
                que.push(node->right);
            }
            else {
                vec.push_back(-1);
            }
        }
        result.push_back(vec);
    }
    for (auto i : result){
        for (auto j : i){
            cout << j << " ";
        }
        cout << endl;
    }
}

// 前序遍历迭代法
vector<int> preorderTraversal(TreeNode* root){
    stack<TreeNode* > st;
    vector<int> result;
    if (root == NULL) return result;
    st.push(root);
    while (!st.empty()) {
        TreeNode* node = st.top();
        st.pop();
        result.push_back(node->val);
        if (node->right) st.push(node->right);
        if (node->left) st.push(node->left);
    }
    return result;
}

// 中序遍历迭代法
vector<int> inorderTraversal(TreeNode* root){
    vector<int> result;
    stack<TreeNode* > st;
    TreeNode* cur = root;
    while (cur != NULL || !st.empty()) {
        if(cur != NULL){
            st.push(cur);
            cur = cur->left;
        } else {
            cur = st.top();
            st.pop();
            result.push_back(cur->val);
            cur = cur->right;
        }
    }
    return result;
}

// 后续遍历迭代法
vector<int> preorderTraversal(TreeNode* root){
    stack<TreeNode* > st;
    vector<int> result;
    if (root == NULL) return result;
    st.push(root);
    while (!st.empty()) {
        TreeNode* node = st.top();
        st.pop();
        result.push_back(node->val);
        if (node->left) st.push(node->left);
        if (node->right) st.push(node->right);
    }
    reverse(result.begin(), result.end()); // 将结果反转之后就是左右中的顺序了
}

vector<int> preorderTraversal(TreeNode* root) {
        vector<int> result;
        traversal(root, result);
        return result;
}

// 递归法
void traversal(TreeNode* cur, vector<int>& vec) {
    if (cur == NULL) return;
    // 交换下面的顺序 分别对应 左中右 遍历
    traversal(cur->left, vec);  // 左
    vec.push_back(cur->val);    // 中
    traversal(cur->right, vec); // 右
}

// 迭代法的统一

vector<int> inorderTraversal(TreeNode* root) {
    vector<int> result;
    stack<TreeNode*> st;
    if (root != NULL) st.push(root);
    while (!st.empty()) {
        TreeNode* node = st.top();
        if (node != NULL) {
            st.pop(); // 将该节点弹出，避免重复操作，下面再将右中左节点添加到栈中
            if (node->right) st.push(node->right);  // 添加右节点（空节点不入栈）
            st.push(node);                          // 添加中节点

            st.push(NULL); // 中节点访问过，但是还没有处理，加入空节点做为标记。

            if (node->left) st.push(node->left);    // 添加左节点（空节点不入栈）
        } else { // 只有遇到空节点的时候，才将下一个节点放进结果集
            st.pop();           // 将空节点弹出
            node = st.top();    // 重新取出栈中元素
            st.pop();
            result.push_back(node->val); // 加入到结果集
        }
    }
    
    return result;
}


int main() {
    vector<int> vec = {4,1,6,0,2,5,7,-1,-1,-1,3,-1,-1,-1,8};
    TreeNode* root = construct_binary_tree(vec);
    print_binary_tree(root);
}
```

## 二叉树解题思路

二叉树解题的思维模式分两类：

- 1、是否可以通过遍历一遍二叉树得到答案？如果可以，用一个 `traverse` 函数配合外部变量来实现，这叫「**遍历**」的思维模式。

- 2、是否可以定义一个递归函数，通过子问题（子树）的答案推导出原问题的答案？如果可以，写出这个递归函数的定义，并**充分利用这个函数的返回值**，这叫「分解问题」的思维模式。

无论使用哪种思维模式，你都需要思考：

**如果单独抽出一个二叉树节点，它需要做什么事情？需要在什么时候（前/中/后序位置）做**？其他的节点不用你操心，递归函数会帮你在所有节点上执行相同的操作。

所谓**前序位置**，就是刚进入一个节点（元素）的时候，**后序位置**就是即将离开一个节点（元素）的时候，那么进一步，你把代码写在不同位置，代码执行的时机也不同：
![](https://labuladong.gitee.io/algo/images/%e4%ba%8c%e5%8f%89%e6%a0%91%e6%94%b6%e5%ae%98/1.jpeg)

前中后序是遍历二叉树过程中处理每一个节点的三个特殊时间点，绝不仅仅是三个顺序不同的 List：

前序位置的代码在刚刚进入一个二叉树节点的时候执行；

后序位置的代码在将要离开一个二叉树节点的时候执行；

中序位置的代码在一个二叉树节点左子树都遍历完，即将开始遍历右子树的时候执行。
![](https://labuladong.gitee.io/algo/images/%e4%ba%8c%e5%8f%89%e6%a0%91%e6%94%b6%e5%ae%98/2.jpeg)

### 例题1

leetcode[104] 二叉树的最大深度。给定一个二叉树，找出其最大深度。

二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。

**解题思路：**

- 遍历的思路：遍历一遍二叉树，用一个外部变量记录每个节点所在的深度，取最大值就可以得到最大深度。
- 二叉树的最大深度可以通过子树的最大深度推导出来。确实可以通过子树的最大深度推导出原树的深度，所以当然要首先利用递归函数的定义算出左右子树的最大深度，然后推出原树的最大深度，主要逻辑自然放在后序位置。

1.

```c++
// 记录最大深度
int res = 0;
// 记录遍历到的节点的深度
int depth = 0;

// 主函数
int maxDepth(TreeNode root) {
	traverse(root);
	return res;
}

// 二叉树遍历框架
void traverse(TreeNode root) {
	if (root == null) {
		return;
	}
	// 前序位置
	depth++;
    if (root.left == null && root.right == null) {
        // 到达叶子节点，更新最大深度
		res = Math.max(res, depth);
    }
	traverse(root.left);
	traverse(root.right);
	// 后序位置
	depth--;
}
```
2.
```c++
// 定义：输入根节点，返回这棵二叉树的最大深度
int maxDepth(TreeNode root) {
	if (root == null) {
		return 0;
	}
	// 利用定义，计算左右子树的最大深度
	int leftMax = maxDepth(root.left);
	int rightMax = maxDepth(root.right);
	// 整棵树的最大深度等于左右子树的最大深度取最大值，
    // 然后再加上根节点自己
	int res = Math.max(leftMax, rightMax) + 1;

	return res;
}

```

