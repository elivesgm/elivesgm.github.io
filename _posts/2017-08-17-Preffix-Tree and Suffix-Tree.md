---
layout: post
title: "Prffix-Tree and Suffix-Tree"
date: 2017-08-17 08:15:06 
description: "Hash Algorithms"
tag: Algorithm
---

## 1. 前缀树
前缀树的3个基本性质：
- 根节点不包含字符，除根节点外每一个节点都只包含一个字符。
- 从根节点到某一节点，路径上经过的字符连接起来，为该节点对应的字符串。
- 每个节点的所有子节点包含的字符都不相同。



## 2. 后缀树
构建后缀树的抽象过程：
- 根据文本 Text 生成所有后缀的集合；
- 将每个后缀作为一个单独的关键词，构建一棵 Compressed Trie。
比如，对于文本 "banana\0"，其中 "\0" 作为文本结束符号。该文本所对应的所有后缀：
- banana\0
- anana\0
- nana\0
- ana\0
- na\0
- a\0
- \0

### References

[1] [后缀树](http://www.cnblogs.com/gaochundong/p/suffix_tree.html),cnblogs.com.

[2] [Trie树|前缀树的介绍与实现](https://songlee24.github.io/2015/05/09/prefix-tree/),github.io.
