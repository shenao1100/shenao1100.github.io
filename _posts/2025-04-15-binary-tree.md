---
title: 二叉树入门
description: 学习笔记
author: ShenNya
date: 2025-04-15 17:07:00 +0800
categories: [Algorithm]
tags: [Algorithm, C++]
math: true
mermaid: true
---

> 如果你看到我写这些这样的笔记了，那一定是我的大脑存不下，而之后这个东西又比较常用

# 二叉树

二叉树是一种树形结构(废话)

它的特点是一个`根(root)`最多有两个`子节点(node)`


**根节点（Root）**：树的起始节点，所有操作都从根节点开

**父节点（Parent）**：某个节点的直接上级节

**子节点（Child）**：某个节点的直接下级节

**叶子节点（Leaf）**：没有子节点的节

**深度（Depth）**：节点到根节点的路径长

**高度（Height）**：从当前节点到最远叶子节点的路径长

```text
      A
    /   \
   B     C
  / \   / \
 D   E F   G
```

喏，二叉树


以`A`为例，`A`为`根`，`B`和`C`为两个`节点`

`A`为`B`的父节点

同时以`B`为根, `D`和`E`为两个`节点`，以此类推


# 种类

## 满二叉树

除了叶子节点（没有子节点的节点）外都有两个子节点的

满二叉树：
```text
      A
    /   \
   B     C
  / \   / \
 D   E F   G

```

**非**满二叉树：
```text
      A
    /   \
   B     C
  / \   / 
 D   E F   

```


## 完全二叉树

1. 除了最后一层外，每层的节点都填满了
2. 最后一层节点


完全二叉树：

```text
      A
    /   \
   B     C
  / \   / \
 D   E F   G

```
```text
      A
    /   \
   B     C
  / \   / 
 D   E F   

```

**非**完全二叉树：

```text
      A
    /   \
   B     C
  / \     \
 D   E     G

```

> 除此之外还有 *平衡二叉树* 和 *二叉搜索树*, 下次一定.jpg

# 遍历

```text
      A
    /   \
   B     C
  / \   / \
 D   E F   G
```

## 先序遍历 (Preorder)

根 → 左 → 右

```
ABDECFG
```

1. 从根出发(A)
2. 先遍历左子树(B)
3. 发现`B`为`D` `E`的父节点
4. 遍历`B`的左子树`D`
5. `D`没有`子节点`, 于是遍历右子树`E`
6. `A`的左子树`B`遍历完毕，开始遍历右子树`C`
7. 以此类推

## 中序遍历 (Inorder)

左 → 根 → 右

```
DBEAFCG
```
## 后序遍历 (Postorder)

左 → 右 → 根

```
DEBFGCA
```
## 层序遍历 (Level-order)

按层从上到下、从左到右访问节点

```
ABCDEFG
```


# C++实现

## 节点

首先实现一个节点，这个节点有一个值，并且有0-2个子节点：

```cpp
struct TreeNode{
    char val;       // 当前节点的值
    TreeNode* left; // 左子节点
    TreeNode* right;// 右子节点
    TreeNode(char v) : val(v), left(nullptr), right(nullptr) {}
};
```

## 创建一个树

我们来创建一个这样的树：
```text
      A
     / \
    B   C
   / \
  D   E
```

```cpp

int main(){
    TreeNode* root = new TreeNode('A');   // 这就是根节点
    root->left = new TreeNode('B');
    root->right = new TreeNode('C');
    root->left->left = new TreeNode('D');
    root->left->right = new TreeNode('E');

    return 0;
}
```

## 先序排列

传入根节点之后按照根左右的顺序递归

```cpp
void preorder(TreeNode* root){
    // 输出根
    cout << root->val;
    if (root->left != nullptr){
        // 递归左子树 
        preorder(root->left);
    }
    if (root->right != nullptr){
        // 递归右子树 
        preorder(root->right);
    }
}
```

## 中序排列

传入根节点之后按照左根右的顺序递归


```cpp
void inorder(TreeNode* root){
    if (root->left != nullptr){
        inorder(root->left);
    }
    cout << root->val;
    if (root->right != nullptr){
        inorder(root->right);
    }
}
```

## 后续排列

传入根节点之后按照左右根的顺序递归

```cpp
void postorder(TreeNode* root){
    if (root->left != nullptr){
        postorder(root->left);
    }
    if (root->right != nullptr){
        postorder(root->right);
    }
    cout << root->val;
}
```

## 层序排列

层序排列不想需要递归，使用队列即可

```cpp
void levelorder(TreeNode* root){
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()){
        TreeNode* node = q.front();
        cout << node->val;
        if (node->left){
            q.push(node->left);
        }
        if (node->right){
            q.push(node->right);
        }
        q.pop();
    }
}
```

## 完整代码

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

void inorder(TreeNode* root){
    if (root->left != nullptr){
        inorder(root->left);
    }
    cout << root->val;
    if (root->right != nullptr){
        inorder(root->right);
    }
}

void postorder(TreeNode* root){
    if (root->left != nullptr){
        postorder(root->left);
    }
    if (root->right != nullptr){
        postorder(root->right);
    }
    cout << root->val;
}

void levelorder(TreeNode* root){
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()){
        TreeNode* node = q.front();
        cout << node->val;
        if (node->left){
            q.push(node->left);
        }
        if (node->right){
            q.push(node->right);
        }
        q.pop();
    }
}
int main(){
    TreeNode* root = new TreeNode('A');
    root->left = new TreeNode('B');
    root->right = new TreeNode('C');
    root->left->left = new TreeNode('D');
    root->left->right = new TreeNode('E');

    preorder(root);
    cout << endl;
    inorder(root);
    cout << endl;
    postorder(root);
    cout << endl;
    levelorder(root);
    return 0;
}
```


