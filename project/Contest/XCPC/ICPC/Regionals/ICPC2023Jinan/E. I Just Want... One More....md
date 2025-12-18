---
title: I Just Want... One More...
date: 2025-11-03
last_clear: 2025-11-03
tags:
  - QOJ/ICPC/Regionals/Jinan/2023
  - SUA
  - tag/图论/二分图
  - tag/图论/网络流/最大流
grasp: 2
value: 5
difficulty:
optional_difficulty: Easy-Medium
source: https://qoj.ac/contest/1472/problem/7898
article:
---
网络流性质：
如果没有达到最大流，则残量网络中一定存在由 $s \rightarrow t$ 的增广路，因此只需要跑最大流后，找到左侧$s$可以"增广"到的地方的数量，右侧可以"增广"到$t$的地方的数量，相乘即为答案
> 若存在增广路，则匹配非最大（Ford–Fulkerson 基本定理）
