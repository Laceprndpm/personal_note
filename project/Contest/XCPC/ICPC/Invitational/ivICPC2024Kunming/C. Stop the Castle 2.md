---
title: Stop the Castle 2
title2: 阻止城堡2
date: 2025-10-31
last_clear: 2025-10-31
tags:
  - QOJ/ICPC/Invitational/Kunming/2024
  - SUA
  - tag/图论/网络流/最大流
  - tag/图论/二分图
grasp: 3
value: 4
difficulty:
optional_difficulty: Hard-Medium
source: https://qoj.ac/contest/1802/problem/9424
article:
---
做过阻止城堡1，所以一开始就往二分图上想了
但是注意到点数 $n \le 10^5$ ，如果对所有对城堡都连边，那么边数 $V = n^2 = 10^{10}$ 无法接受。
然后我发现， $m \le 1e5$ ，代表我们放城堡只会放在这 $1e5$ 个点上，那么思路就很明确了，对于这个二分图，直接跑一遍二分图最大匹配，然后从劣到优排序，删除前 $k$ 个劣的即可
因为优的事先确定了，所以不会有依赖性关系
