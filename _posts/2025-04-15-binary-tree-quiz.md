---
title: 二叉树入门的练习
description: 学习笔记
author: ShenNya
date: 2025-04-15 21:47:00 +0800
categories: [Algorithm]
tags: [Algorithm, C++]
math: true
mermaid: true
---


# P1030 [NOIP 2001 普及组] 求先序排列

[洛谷题源](https://www.luogu.com.cn/problem/P1030)

## 题目描述

给出一棵二叉树的中序与后序排列。求出它的先序排列。（约定树结点用不同的大写字母表示，且二叉树的节点个数 $ \le 8$）。

## 输入格式

共两行，均为大写字母组成的字符串，表示一棵二叉树的中序与后序排列。

## 输出格式

共一行一个字符串，表示一棵二叉树的先序。

## 输入输出样例 #1

### 输入 #1

```
BADC
BDCA
```

### 输出 #1

```
ABCD
```

## 说明/提示

**【题目来源】**

NOIP 2001 普及组第三题

---

## 思路

1. `后序排列` 的最后一个元素是 `根节点`
2. 在 `中序排列` 中找到该节点的位置，位置的左边为剩下的左子树，位置的右边为剩下的右子树
3. 根据 `中序排列` 划分的长度，在 `后续排列` 中也划分出相应的左子树和右子树
4. 递归处理

```cpp
#include <iostream>
#include <queue>

using namespace std;

struct TreeNode{
    char val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(char v) : val(v), left(nullptr), right(nullptr) {}
};

void preorder(TreeNode* root){
    cout << root->val;
    if (root->left != nullptr){
        preorder(root->left);
    }
    if (root->right != nullptr){
        preorder(root->right);
    }
}


TreeNode* process(string inorder_str, string postorder_str){
    // 如果字符串空了就返回null
    if (inorder_str.empty() || postorder_str.empty()) {return nullptr;}



    // 寻找根
    char root_val = postorder_str[postorder_str.size() - 1];
    // 建立节点
    TreeNode* root = new TreeNode(root_val);

    // 获取中序 后序的左右部分
    string left_in, right_in, left_post, right_post;
    // 寻找根在中序的位置
    long root_posi = inorder_str.find(root_val);
    // 后序排列，相当于 left_post, right_post = postorder_str.split(root_val)
    left_post = postorder_str.substr(0, root_posi);
    right_post = postorder_str.substr(root_posi, postorder_str.size() - root_posi - 1);
    // 中序排列，相当于left_in = inorder_str[:len(root_posi)]; right_in = inorder_str[len(root_posi):]
    left_in = inorder_str.substr(0, root_posi);
    right_in = inorder_str.substr(root_posi + 1, inorder_str.size() - root_posi - 1);

    root->left = process(left_in, left_post);
    root->right = process(right_in, right_post);
    return root;
}

int main(){
    string inorder_str, postorder_str;

    cin >> inorder_str >> postorder_str;

    TreeNode* root = process(inorder_str, postorder_str);
    preorder(root);

    return 0;
}

```



