---
title: U - Grouping
date: 2025-07-09
last_clear: 2025-06-30
tags:
  - tag/dp
  - tag/bitmasks
grasp: 3
difficulty: 
source: https://atcoder.jp/contests/dp/tasks/dp_u
article: https://oi-wiki.org/math/binary-set/
article1: https://www.luogu.com.cn/problem/solution/AT_dp_u
---
比较板的 sum of sons，可以参考article里的oi-wiki补一下子集遍历

子集遍历的复杂度为 $O(n^3)$ 
>我们可以将不属于 mask 的元素看作 0，属于 mask 但不属于 pmask 的元素看作 1，属于 pmask 的元素看作 2。对于每个兔子，都有 3 种可能的状态，所以总时间复杂度是 $O(n^3)$。
