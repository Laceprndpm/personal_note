---
title: Japanese Bands
date: 2025-10-28
last_clear: 2025-10-28
tags:
  - QOJ/ICPC/Regionals/Hangzhou/2024
  - SUA
  - tag/bitmasks
  - tag/计数
  - tag/thinking/约束分析
  - tag/dp/sosdp
  - tag/math
grasp: 1
value: 5
difficulty:
optional_difficulty: Easy-Medium
source: https://qoj.ac/contest/1893/problem/9735
article: https://qoj.ac/download.php?type=attachments&id=1893&r=1
---
# Learnt
- 当出现限制$\{a_i, b_i\}$ 是不同集合时，可以直接分成三个集合， $A$ 独占、 $B$ 独占、均有
- 当暴力做法需要遍历 $\{\text{for i in range(1 << m): for j in range(1 << m)}\}$ ，然后再用约束检查时，不如直接尝试遍历子集/补集的子集，再尝试用 `sosdp` 优化，即约束分析
# Solution
```cpp showLineNumbers
Z binom(int n, int m)
{
	if (n < m || m < 0) return 0;
	Z res = 1;
	for (int i = 0; i < m; i++) {
		res *= (n - i);
	}
	return res * invfac(m);
	// return fac(n) * invfac(m) * invfac(n - m);
}
```

```cpp showLineNumbers
void solve()
{
    int n1, n2, m, k;
    cin >> n1 >> n2 >> m >> k;
    vector<pair<int, int>> constraints(k);
    // 对于每种蓝色的方案数，计算出红色满足他至少要多少张卡片，其他的随意即可？
    // 复杂度上界为20'9715'2000，3秒，可以勉勉强强?
    // 无法确定红色所需要的卡片种类
    // 也无法确定他必然不能缺少哪些卡牌
    //
    // ===============================
    // 可以只枚举补集的子集，作为B独享的数字，同时通过k中约束来约束出
    // 然后用sos预处理交
    //
    int maskM = 0;
    for (int i = 0; i < k; i++) {
        cin >> constraints[i].fi >> constraints[i].se;
        constraints[i].fi--;
        constraints[i].se--;

        maskM |= 1 << constraints[i].fi;
        maskM |= 1 << constraints[i].se;
    }
    const int M = __builtin_popcount(maskM);
    vector<Z> m2dp(1 << m);
    for (int m2 = 0; m2 < (1 << m); m2++) {
        bool ok = 1;
        if (m2 & (((1 << m) - 1) ^ maskM)) continue;
        for (auto [a, b] : constraints) {
            if (m2 >> a & 1 && m2 >> b & 1) {
                ok = 0;
                break;
            };
        }
        if (!ok) continue;
        m2dp[m2] = comb.binom(n1 - 1 + m - M, m - __builtin_popcount(m2) - 1);
    }
    for (int i = 0; i < m; i++)
        for (int sta = (1 << m) - 1; sta >= 0; sta--)
            if (sta & (1 << i)) m2dp[sta] += m2dp[sta ^ (1 << i)];
    Z ans = 0;
    for (int m1 = 0; m1 < (1 << m); m1++) {
        bool ok = 1;
        if (m1 & (((1 << m) - 1) ^ maskM)) continue;
        for (auto [a, b] : constraints) {
            if (m1 >> a & 1 && m1 >> b & 1) {
                ok = 0;
                break;
            };
        }
        if (!ok) continue;
        ans += comb.binom(n2 - 1 + m - M, m - __builtin_popcount(m1) - 1) * m2dp[(((1 << m) - 1) ^ m1)];
    }
    cout << ans << '\n';
}
```
- 设限制条件里共出现 $M$ 种数，这 $M$ 种数可以分成两类：
  一类在两种卡片里都要出现，一类恰出现于一种卡片里。
- 设 $m_1$ 表示只出现在红卡上的数的二进制 mask，
  $|m_1|$ 表示这个 mask 的大小。相似地，设 $m_2$ 表示只出现在蓝卡上的数的二进制 mask。
  那么剩下的 $(M - |m_1| - |m_2|)$ 种数必须在两种卡片上都出现。
  而未在限制条件里出现的 $(m - M)$ 种数则可出现可不出现。
  因此答案为：

$$
\sum_{m_1} \sum_{m_2 \cap m_1 = \varnothing}
f_1(m_1, m_2) \times f_2(m_1, m_2)
$$
其中：
$$
f_1(m_1, m_2)
= \binom{
n_1 - 1 + (m - M)
}{
|m_1| + (M - |m_1| - |m_2|) - 1 + (m - M)
}
$$
$$
f_2(m_1, m_2)
= \binom{
n_2 - 1 + (m - M)
}{
|m_2| + (M - |m_1| - |m_2|) - 1 + (m - M)
}
$$
化简一下，答案为：

$$
\sum_{m_1} \sum_{m_2 \cap m_1 = \varnothing}
\binom{n_1 - 1 + (m - M)}{m - |m_2| - 1}
\times
\binom{n_2 - 1 + (m - M)}{m - |m_1| - 1}
$$
等价于：
$$
\sum_{m_1}
\binom{n_2 - 1 + (m - M)}{m - |m_1| - 1}
\sum_{m_2 \cap m_1 = \varnothing}
\binom{n_1 - 1 + (m - M)}{m - |m_2| - 1}
$$
因此可以用高维前缀和预处理：
$$
\sum_{m_2 \cap m_1 = \varnothing}
\binom{n_1 - 1 + (m - M)}{m - |m_2| - 1}
$$
再枚举 $m_1$ 即可。
另外，$m_1$ 里的每对元素不能出现在同一个限制条件里，$m_2$ 同理。
因为 $m$ 只有 $20$，所以可以用一个 `int` 把和每个数有关的限制条件存下来，
枚举 $m_1$ 和 $m_2$ 时通过位运算检验一下合法性即可。
复杂度为 $O(m \times 2^m)$。
