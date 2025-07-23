---
title: H - Head out to the Target
date: 2025-07-22
last_clear: 2025-07-22
tags:
  - tag/trees/树上倍增
grasp: 3
difficulty: 1700
source: https://ac.nowcoder.com/acm/contest/108300/H
article:
---
写了错解，用dfsn序+线段树
正解为树上倍增
>观察到任意时刻可达的点是一个包含根的极大连通块。
>考虑直接维护这个连通块，当目标在 $[r_i, l_i]$ 时刻内出现在 $u_i$ 时，相当于
将这个连通块向 $u_i$ 方向扩张 $(r_i − l_i + 1)$ 步。
>考虑使用倍增，从 ui 开始向上找到最接近的可达点，然后暴力向下逐
步标记为可达，若在 $(r_i − l_i + 1)$ 步前 $u_i$ 已经被标记可达即可输出答案。
