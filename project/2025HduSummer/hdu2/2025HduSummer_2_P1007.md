---
title: 计数
date: 2025-07-21
last_clear: 2025-08-16
tags:
  - mystery
  - tag/dp/计数
  - tag/math
  - tag/math/二项式基底
  - tag/math/拉格朗日插值
grasp: 1
difficulty: 2400
optional_difficulty: Medium
source: https://acm.hdu.edu.cn/contest/problem?cid=1173&pid=1007
article:
---
# 题意
给定正整数序列 $p_1, \; p_2 \dots p_n$ 和正整数 $R$ ，求有多少正整数序列 $a_1, \dots , a_n$ 满足：
- $p_i \le a_i \le R$
- $a_1 \ge a_2 \ge \dots \ge a_n$
答案对 $1e9 + 7$ 取模
# Solution
## std 二项式基底
设 $M_i$ 表示 $a_i$ 到 $a_n$ 的最大值。不难发现，可以将 $a_i$ 替换成 $M_i$。

设 $F_i(x)$ 表示：已经考虑了 $b_i$ 到 $b_n$，且 $b_i = x$ 时的方案数。显然有递推关系：

$$
F_i(x) = \sum_{y = M_{i+1}}^{x} F_{i+1}(y)
$$

注意到 $F_i(x)$ 相当于是 $F_{i+1}(x)$ 的前缀和。不难发现 $F_i(x)$ 是关于 $x$ 的 $n - i$ 次多项式。

考虑将 $F_i(x)$ 写成“二项式系数基”的形式：

$$
F_i(x) = \sum_{k=0}^{n-i} c_{i,k} \binom{x}{k}
$$

代入递推式，得

$$
F_i(x) = \sum_{y = M_{i+1}}^{x} \sum_{k=0}^{n-i-1} c_{i+1,k} \binom{y}{k} = \sum_{k=0}^{n-i-1} c_{i+1,k} \sum_{y = M_{i+1}}^{x} \binom{y}{k}
$$

利用组合恒等式，有

$$
\sum_{y = M_{i+1}}^{x} \binom{y}{k} = \binom{x + 1}{k + 1} - \binom{M_{i+1}}{k + 1}
$$

进一步展开，可得

$$
F_i(x) = \sum_{k=0}^{n-i-1} c_{i+1,k} \cdot \left[ \binom{x + 1}{k + 1} - \binom{M_{i+1}}{k + 1} \right]
$$

或等价地写成

$$
F_i(x) = \sum_{k=0}^{n-i-1} c_{i+1,k} \cdot \left[ \binom{x}{k + 1} + \binom{x}{k} - \binom{M_{i+1}}{k + 1} \right]
$$

因此，可以在 $O(n)$ 时间内从 $F_{i+1}(x)$ 的系数递推得到 $F_i(x)$ 的系数。

在计算答案时，可以令 $a_0 = R$，最终的 $F_0(R)$ 即为所求答案。

**时间复杂度**：$O(n^2)$。
```c++
const int N = 5010;

const int mod = 1e9 + 7;

int qpow(int a, int b, int p)
{
    int ans = 1;
    for (; b; b >>= 1) {
        if (b & 1) ans = 1ll * ans * a % p;
        a = 1ll * a * a % p;
    }
    return ans;
}

int n, R;
int a[N];

int inv[N];

int c[N][N];

void work()
{
    std::cin >> n >> R;

    a[0] = R;
    for (int i = 1; i <= n; i++) {
        std::cin >> a[i];
    }

    for (int i = n - 1; i >= 0; i--) {
        chmax(a[i], a[i + 1]);
    }

    for (int i = 1; i <= n; i++) {
        inv[i] = qpow(i, mod - 2, mod);
    }

    c[n][0] = 1;
    for (int i = n - 1; i >= 0; i--) {
        for (int k = 0; k <= n - i; k++) {
            c[i][k] = 0;
        }

        int b = 1;  // 将从(a_{i+1}, 0)，迭代到(a_{i+1}, k)
        // b就是题解里的(M_{i+1}, k + 1)
        for (int k = 0; k <= n - i - 1; k++) {
            b = 1ll * b * (a[i + 1] - k) % mod;  //
            b = 1ll * b * inv[k + 1] % mod;

            c[i][k + 1] = (c[i][k + 1] + c[i + 1][k]) % mod;  // 第一项
            c[i][k]     = (c[i][k] + c[i + 1][k]) % mod;      // 第二项
            c[i][0]     = (c[i][0] + 1ll * (mod - b) * c[i + 1][k]) % mod;
        }
    }

    int ans = 0, b = 1;
    for (int k = 0; k <= n; k++) {
        ans = (ans + 1ll * b * c[0][k]) % mod;

        b = 1ll * b * (R - k) % mod;
        b = 1ll * b * inv[k + 1] % mod;
    }

    std::cout << ans << '\n';
}
```
[[1972E]] 似乎也是差不多的？
组合数上，与 [[2060F]] 也有相似性

---
## Lagrange
设 $f(i, x)$ 为 $a_i = x$ 时的方案数。
则有递推公式
$$
f(i, x) = \sum_{j = p_{i + 1}}^x f(i + 1, j)
$$
则 $i = n$ 时，$f(n, x) = 1$ 为 $0$ 次多项式。
由递推公式可得，$f(i, x)$ 是 $n - i$ 次多项式。
那么我就可以求
$$
f(i, 0), \ f(i,1), \ f(i,2) \dots f(i, n - i)
$$
来得到这个多项式
#### Trick:
我们求解这些点时，他可能并不是dp值，但是他是dp的值所在函数的真正的$n - i + 1$个点，可以唯一确定这个多项式。
>个人理解指的是 $j<p_{i+1} - 1$ 依然有意义，且不能简单的依赖上文的递推关系直接截断，否则会破坏多项式的连续性。
$$
f(i, j) = S(i + 1, j) - S(i + 1, p_{i + 1} - 1)
$$

其中
$$
S(i, x) = \sum_{j = 1} ^xf(i, j)
$$
对于 $S(i + 1, j)$ 可以直接累加求出
对于 $S(i + 1, p_{i+1} - 1)$ ，考虑到其是一个 $n - i + 1$ 次多项式，依然由特殊情况 2 来 $O(n)$ 求解 $p_{i+1} - 1$ 处的点值。
注意：
由于 $f(i, j)$ 是 $n - i$ 次多项式，不可以这样：
```c++
for (int i = 1; i <= n + 1; i++) {
	if (i >= ai[i]) f_i[1][i] = 1;
	else f_i[1][i] = 0;
}
```
相当于破坏了连续性。

`lagrange` 插值复杂度是 $O(n \log P)$ 的，其中 $P$ 是模数，需要控制常数。
**时间复杂度**： $O(n^2 \log P)$ 
```c++
Z lagrange(int x, Z ptr[], int n)
{
    if (x <= n) {
        return ptr[x];
    }
    Z wi = 1;
    for (int j = 1; j <= n; j++) {
        wi *= Z(x - j);
    }
    Z ans = 0;
    for (int i = 1; i <= n; i++) {
        ans += ptr[i] * wi / (x - i) * comb.invfac(i - 1) * comb.invfac(n - i) * ((n - i) & 1 ? -1 : 1);
    }
    return ans;
}

void solve()
{
    i64 n, R;
    cin >> n >> R;
    vector<int> ai(n + 1);
    for (int i = 1; i <= n; i++) {
        cin >> ai[i];
    }
    for (int i = n - 1; i >= 1; i--) {
        ai[i] = max(ai[i + 1], ai[i]);
    }
    vector<Z> f_i[2];
    f_i[0].resize(n + 2);
    f_i[1].resize(n + 2);
    for (int i = 1; i <= n + 1; i++) {
        f_i[1][i] = 1;
    }
    vector<Z> S(n + 2);
    for (int i = n - 1; i >= 0; i--) {
        for (int j = 1; j <= n + 1; j++) {
            S[j] = S[j - 1] + f_i[1][j];
        }
        Z S2 = lagrange(ai[i + 1] - 1, S.data(), n - i + 1);
        for (int j = 1; j <= n + 1; j++) {
            f_i[0][j] = S[j] - S2;
        }
        swap(f_i[0], f_i[1]);
    }
    Z ans = lagrange(R, f_i[1].data(), n + 1);
    cout << ans << '\n';
}
```
