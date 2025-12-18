---
title: 夜世界
date: 2025-07-18
last_clear: 2025-07-18
tags:
  - tag/主席树
  - tag/模拟
grasp: 2
difficulty: 1800
source: https://acm.hdu.edu.cn/contest/problem?cid=1172&pid=1002
article:
---
因为有查询4，因此我们要找到一个区间合并的方法
1. 当前的贡献值不能少
2. 我还剩多少钱，用于对被打劫时进行计算
3. 因为如果我以带钱状态进入与空手进入不同，因此我们要记录我们应给但没有给，也就是"哥布林看我们可怜"时欠的钱。
随后的区间合并相当直觉
```c++
    friend Info operator+(const Info& a, const Info& b)
    {
        Info c;
        c.contri = a.contri + b.contri + min(b.need, a.remain);
        c.need   = a.need + b.need - min(b.need, a.remain);
        c.remain = a.remain + b.remain - min(b.need, a.remain);
        return c;
    }
```
值得注意的是，主席树的空间应该开 $4 \cdot n + m\cdot \text{log}(n)$ 其中 $n$ 是节点数， $m$ 是操作数。
```c++
	int mem = (4 << __lg(n_)) + (m_)*ceil_log2(n_);
```
