---
title: W - Intervals
date: 2025-07-09
last_clear: 2025-07-09
tags:
  - tag/dp
  - tag/SegmentTree
grasp: 1
difficulty: 
source: https://atcoder.jp/contests/dp/tasks/dp_w
article: https://www.luogu.com.cn/article/dwvivxm6
---
对于区间的处理比较巧妙：所有的贡献在右端点处理
不需要考虑倒数第二个节点，而是设 $dp[\_i][j]$ (滚动优化dp)，考虑前 $\_i$个节点的贡献时，最后一个节点在 $j$ 位置的最大价值
那么转移时，转移到 $dp[i][i]$ 时用线段树/树状数组来维护区间最大值，直接从最大值转移
贡献计算，只需要把所有 $l_i \le j$ 的贡献加上即可
