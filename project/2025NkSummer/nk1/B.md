---
title: Binary Substrings 2
date: 2025-09-01
last_clear: 2025-09-01
tags:
  - tag/constructive_algorithms
  - tag/估算
  - tag/math
grasp: 1
difficulty:
source: https://ac.nowcoder.com/acm/contest/108298/B
article:
---
# 题意
给定两个整数 n 和 m，你需要找到一个长为 n 的二进制字符串，其本质不同的非空子串的数量恰好为 m。这里 m 不小于 n，(长为 n 的二进制字符串中本质不同非空子串的最小数量)，并且不超过$M_n = \sum\min\{2^i, n - i + 1\}$ (这是已经得到证明的最大数量)。
然而上述这个问题似乎太难了，因此你只需要找到一个长为 n 的二进制字符串，使其不同的非空子字符。
串的数量与 m 的相对误差最多为 0.2，即在范围 $[0.8 \times m, 1.2 \times m]$ 内，或者指出没有满足条件的二进制
字符串。
# Solution
## std
由于不需要精确地构造，可以简单认为 $M_n$ 是在 $n^2/2$ 左右。构造方式可能有很多，以下给出的是标程使用的构造方式。
一个很容易想到的构造方式是 `k` 个 `0` 后接 `n − k` 个 `1`，这样能得到 $k(n − k) + n$ 个本质不同子串，具体值依次是 $n, 2n − 1, 3n − 4, \dots$
这个构造的最大值在 $n^2 / 4$ 左右，可以覆盖到 $5n^2 / 16$ 左右，并且除了 $n$ 和 $2n - 1$ 之间有一段覆盖不到，其他相邻两个值之间都是完全覆盖的。
容易证明当 `n > 1` 时 `2n − 1` 是次小值，那么 $[n, 2n − 1]$ 里可能会有一段无解，判掉即可。

接下来考虑覆盖 $5n^2/16$ 左右到 $n^2/2$ 左右的部分。
考虑构造 `t` 段 0，每段长度都在 $n/t$ 左右，第 i 段和第 i + 1 段之间用 `i`个 1 分隔，这样跨过 1 的子串都本质不同，可以得到 $\frac{t - 1}{2 t} n^2$ 左右。
t 从 $3$ 取到 $6$ 就基本够了，需要注意 n 比较小的时候可能有边界问题。
如果对估计值不够放心，可以使用后缀数据结构计算构造方案的精确值。
## Mysolution
理解了std，认为其情况二的构造应该是尽量使得每一个 $[i, j]$ 都是不同的，而方法就是插入几个间断点（1段），并且让每个间断点都不一样。由于1数量<<0数量，所以可以这么近似。
```cpp
inline void NO()
{
    cout << -1 << '\n';
}

void case1(int n, i64 m)
{
    for (i64 k = 0; k < n; k++) {
        if ((k * (n - k) + n) * 5ll < m * 4ll) continue;
        cout << string(k, '1') + string(n - k, '0') << '\n';
        return;
    }
    // cout << __LINE__ << '\n';
    assert(0);
}

inline bool check(i64 num, i64 m)
{
    return abs(num - m) * 5 < m;
}

void case2(i64 n, i64 m)
{
    for (i64 k = 2; k <= 7; k++) {
        i64 num = (k - 1) * n * n / (2 * k);
        if (!check(num, m)) continue;
        i64 rem  = n - (k - 1) * k / 2;
        i64 each = rem / k;
        for (int i = 0; i < k; i++) {
            cout << string(each, '0');
            if (i != k - 1) {
                cout << string(i + 1, '1');
            }
        }
        //
        cout << string(rem - each * k, '0');
        cout << '\n';
        return;
    }
    // SHOW(1);
    assert(0);
}

void solve()
{
    i64 n, m;
    cin >> n >> m;
    int k = n / 2;
    if (m * 4 > n * 5 && 6 * m < (2 * n - 1) * 5) NO();
    // n < m *  0.8, m * 6 < 10 * n - 5
    else if ((m * 4 <= n * 5 || 6 * m >= 5 * (2 * n - 1)) && (5 * ((n - k) * k + n) >= 4 * m))
        case1(n, m);
    else
        case2(n, m);
    // k = (n - 1) / 2
    // k * (n - k) = n * k - k * k
    // k = (n - 1) / 2, n / 2
    // k(n − k) + n
    //
    // n, 2n - 1, 3n - 4, 4n - 9
    //
    // (2.5n - 2.5) * 0.8 = 2n - 2
    //  3n - 3
    //
    // (n - k) * k + n > 0.8 * m
    //
    // n * n / 4 / (4 / 5) = 5 / 16 = 0.3125 ~ 3 / 10
    //
    //
}

signed main(signed argc, char** argv)
{
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
#ifdef BATCH
    freopen(argv[1], "r", stdin);
    freopen(argv[2], "w", stdout);
#endif
    int t;
    cin >> t;
    while (t--) {
        solve();
    }
    return 0;
}

/* stuff you should look for
 * int overflow, array bounds
 * special cases (n=1?)
 * do smth instead of nothing and stay organized
 * WRITE STUFF DOWN
 * DON'T GET STUCK ON ONE APPROACH
 */
```
