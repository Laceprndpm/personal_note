---
title: X - Tower
date: 2025-07-09
last_clear: 2025-07-09
tags:
  - tag/dp
  - tag/贪心
grasp: 1
difficulty: 
source: https://atcoder.jp/contests/dp/tasks/dp_x
---
贪心+dp
当选择的箱子确定，如果 $box_i$ 放在 $box_j$ 上更优，需满足
$min(s_i, s_j - w_i) \ge min(s_j, s_i - w_j)$ 即:
$min(s_i, s_j, s_j - w_i) \ge min(s_i, s_j, s_i - w_j)$ 
所以只要
$s_j - w_i \ge s_i - w_j$ 即可

随后从小到大进行dp， $dp[i]$ 为当重量为 $i$ 时的最高价值
