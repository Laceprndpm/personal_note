---
title: Function Query
date: 2025-10-19
last_clear: 2025-10-20
tags:
  - QOJ/ICPC/Regionals/Asia_East_Continent/Sichuan/2024
  - tag/数据结构/字典树
  - tag/二分搜索
  - tag/math
  - WHO/UESTC
grasp: 4
value: 3
difficulty:
optional_difficulty: Easy-Medium
source: https://qoj.ac/contest/1862/problem/9669
article:
---
其实用介值定理，二分即可，赛时是用min-max乱搞过去的
# Solution
01-tire存储下标，找到大于b和小于b的，直接二分区间
**二分不需要单峰，三分才需要**
