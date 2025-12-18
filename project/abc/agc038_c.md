---
title: C - LCMs
date: 2025-08-24
last_clear: 2025-08-24
tags:
  - tag/math/数论/莫比乌斯反演
grasp: 2
difficulty: 2100
source: https://atcoder.jp/contests/agc038/tasks/agc038_c
article:
---
# 题意
给定序列$A_0 \dots A_{n - 1}$ ，求解所有的 $0 \le i < j < n, \sum{\mathrm{lcm} (A_i, A_j)}$ 
# Solution
$$
\begin{align}
& \sum_{i=0}^{N} \sum_{j=0}^{N} \mathrm{lcm}(A_i, A_j) \\
& \quad = \sum_{i=0}^{N} \sum_{j=0}^{N} \frac{A_i A_j}{\gcd(A_i, A_j)} \\
& \quad = \sum_{d=1}^{L} \frac{1}{d} \sum_{i=0}^{N} \sum_{j=0}^{N} A_i A_j \cdot [\gcd(A_i, A_j) = d]
\quad \text{where} \quad L = \max_{z=0}^{N} (A_z)
\end{align}
$$

$$
\begin{align}
f(d) &= \sum_{i=0}^{N} \sum_{j=0}^{N} A_i A_j \cdot [\gcd(A_i, A_j) = d] \\
g(d) &= \sum_{d \mid d'} f(d') 
= \left( \sum_{i=0}^{N} A_i \cdot [d \mid A_i] \right)^2 \\
f(d) &= \sum_{d \mid d'} \mu\left(\frac{d'}{d}\right) g(d') \\
&= \sum_{d \mid d'} \mu\left(\frac{d'}{d}\right) \left( \sum_{i=0}^{N} A_i \cdot [d' \mid A_i] \right)^2
\end{align}
$$

$$
\begin{align}
& \sum_{i=0}^{N} \sum_{j=0}^{N} \mathrm{lcm}(A_i, A_j) \\
& \quad = \sum_{d=1}^{L} \frac{1}{d} \sum_{d|d'} \mu\left(\frac{d'}{d}\right)\left( \sum_{i=0}^{N} A_i \cdot [d' \mid A_i] \right)^2
\end{align}
$$
设:
$$
h(x) = \sum_{i=0}^{N} A_i \cdot [d' \mid A_i]
$$
则可以用桶计数的方法，$O(n + L \log L)$ 的时间复杂度求解所有 $h(x)$ ，随后的$G(d) = \sum_{d|d'} \mu\left(\frac{d'}{d}\right)h(d')^2$ 又可以 $O(L \log L)$ 求出。
最终 $O(n)$ 求得 $\sum i \cdot G(i)$ 为答案。
```cpp showLineNumbers
constexpr i64   MAXN = 1e6;
OUniversal<int> mu(MAXN, mu_calc);

int mu_calc(int p, int exp)
{
    if (exp > 1) {
        return 0;
    } else {
        return -1;
    }
}

void solve()
{
    int n;
    cin >> n;
    vector<i64> ai(n);
    for (int i = 0; i < n; i++) {
        cin >> ai[i];
    }
    vector<int> cnt(MAXN + 2);
    vector<Z>   f1(MAXN + 2);
    for (int i = 0; i < n; i++) {
        cnt[ai[i]]++;
    }
    for (int d = 1; d <= MAXN; d++) {
        for (int k = d; k <= MAXN; k += d) {
            f1[d] += cnt[k] * Z(k);
        }
    }
    for (int i = 1; i <= MAXN; i++) {
        f1[i] *= f1[i];
    }
    Z step1_ans{};
    for (int i = 1; i <= MAXN; i++) {
        Z tmp{};
        for (int j = i; j <= MAXN; j += i) {
            tmp += mu.f_1[j / i] * f1[j];
        }
        step1_ans += tmp / i;
    }
    for (int i = 0; i < n; i++) {
        step1_ans -= ai[i];
    }
    step1_ans /= 2;
    cout << step1_ans << '\n';
}
```
