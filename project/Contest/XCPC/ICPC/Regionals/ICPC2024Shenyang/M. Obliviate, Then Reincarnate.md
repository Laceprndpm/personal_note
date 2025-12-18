---
title: Obliviate, Then Reincarnate
date: 2025-10-18
last_clear: 2025-10-18
tags:
  - tag/图论/有向图
  - tag/图论/连通性相关/SCC
  - tag/图论/有向图/环
grasp: 2
value: 5
difficulty:
optional_difficulty: Easy-Medium
source: https://codeforces.com/gym/105578/problem/M
article:
---
# Solution
## 赛时思路
找到所有的点，该点可以到达某个非0的环，一开始想通过精心构造dfs，来实现答案统计，后发现不可能。
# 正解
实际上，注意到有向图强联通分量的性质，环一定在一个强联通分量内，因此可以构造
