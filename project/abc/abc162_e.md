---
title: E - Sum of gcd of Tuples (Hard)
date: 2025-08-24
last_clear: 2025-08-24
tags:
  - tag/math/数论/莫比乌斯反演
grasp: 2
difficulty: 2200
source: https://atcoder.jp/contests/abc162/tasks/abc162_e
article:
---
# 题意
计算
$$
\sum_{a_1 = 1}^{k} \dots\sum_{a_N = 1}^{k} \gcd(a_1\dots a_N)
$$
# Solution
计算
$$
\sum_{d | k} d \cdot \sum_{a_1 = 1}^{k} \dots\sum_{a_N = 1}^{k} \gcd[a_1 \dots a_N = d] = \sum_{d | k} d \cdot f(d)
$$
可以设
$$
g(d') = \sum_{d' | d} f(d) = {\left\lfloor \frac{k}{d'} \right\rfloor} ^N
$$
则
$$
f(d) = \sum_{d | d'} \mu \left(\frac{d'}{d} \right)g(d')
$$
最终：
$$
\begin{align}
ans &= \sum_{d=1}^kd \sum_{d|d'}\mu\left(\frac{d'}{d}\right) \left\lfloor\frac{k}{d'}\right\rfloor^N \\
&= \sum_{d'=1}\left\lfloor\frac{k}{d'}\right\rfloor^N\sum_{d|d'}d\cdot\mu\left(\frac{d'}{d}\right) \\
&= \sum_{d'=1}^k \left\lfloor\frac{k}{d'}\right\rfloor^N \varphi(d')
\end{align}
$$

重要恒等式:
$$
\boxed{
\sum_{d \mid n} d \cdot \mu\left( \frac{n}{d} \right) = \varphi(n)
}
$$
```cpp
i64 eular_calc(i64 p, i64 exp)
{
    return power(p, exp - 1) * (p - 1);
}

void solve()
{
    int n, k;
    cin >> n >> k;
    OUniversal<i64, decltype(eular_calc)> phi(k, eular_calc);
    auto                                 &phvec = phi.f_1;
    Z                                     ans{};
    for (int i = 1; i <= k; i++) {
        ans += power(Z(k / i), n) * phvec[i];
    }
    cout << ans << '\n';
}
```
