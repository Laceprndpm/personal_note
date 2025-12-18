---
title: 光线折射
date: 2025-07-25
last_clear: 2025-07-25
tags:
  - tag/math/combinatorics
grasp: 2
difficulty: 1800
optional_difficulty: Easy-Medium
source: https://acm.hdu.edu.cn/contest/problem?cid=1174&pid=1003
article:
---
# 题目描述:
计算从(-0.5, 0)水平发出的一道激光经过多次反射/折射后水平射在(n + 0.5, m) 处的总强度。平面内每一点都有一板，会同时发生折射和反射。给定折射系数a和反射系数b，每次折射和反射光的强度需乘以对应系数。反射时，其方向由上改为右，右改为上，折射则不变。
# 正文:
很经典的题目，有多种路径到达终点，所以
$$
\sum_PWeight(P)\cdot Count(P)
$$
同时， `折射次数+反射次数= n + m + 1` ，所以直接枚举反射次数就可以计算出答案。
## try1:
认为反射在路径序列中的位置只受到第一个反射前和最后一个反射后的折射和限制，后发现如果只限制这两个不够。如果把反射看作切换，很容易出现某个方向上的折射次数大于它的上限。
## try2:
直接考虑它在哪些横坐标和纵坐标上发生了反射
如图为只考虑横坐标选择。
![[Pasted image 20250801215430.png]]
显而易见的横纵坐标不相关，可以直接相乘。
之后频繁WA无法调出。对拍后发现没有特判0

**注意特判0**
即代码
```c++
    Z   a2_inv = power(a * a, MOD - 2);
    Z   cur_a  = power(a, j);
    for (int i = 2; (i / 2 <= min(n + 1, m)); i += 2) {
        cur_a *= a2_inv;
        ans += ((i == n + m + 1) ? 1 : cur_a)...
    }
```
最好的方法还是jiangly那样:
```c++
    std::vector<int> pa(n + m + 2), pb(n + m + 2);
    pa[0] = 1;
    pb[0] = 1;
    for (int i = 1; i <= n + m + 1; i++) {
        pa[i] = i64(pa[i - 1]) * a % P;
        pb[i] = i64(pb[i - 1]) * b % P;
    }
```
