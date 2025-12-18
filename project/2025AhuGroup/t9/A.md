---
title: " A. 乘积，欧拉函数，求和"
date: 2025-10-03
last_clear: 2025-10-04
tags:
  - tag/math/分块
  - tag/dp
grasp: 2
value: 5
difficulty:
optional_difficulty: Silver
source: https://qoj.ac/contest/1840/problem/9619
article: https://zhuanlan.zhihu.com/p/16058248745
---
很好的一道题，欧拉函数并非总是对应着推式子或者容斥反演之类的，而是也可以像普通的计数、dp一样
# Solution
```cpp ShowLineNumbers title = "std.cpp"
#include <bits/stdc++.h>
#define up(a, b, c) for (int a = b; a <= c; ++a)
#define dn(a, b, c) for (int a = b; a >= c; --a)
constexpr int N = 2200, q[] = {2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47}, p = 998244353;
using namespace std;
int n, a[N], b[N], c[N], f[1 << 15], g[1 << 15], as;

int ni(int n) { return n - 1 ? p - 1ll * p / n * ni(p % n) % p : 1; }

int main()
{
    cin >> n;
    up(i, 1, n)
    {
        cin >> a[i], b[i] = a[i];
        up(j, 0, 14) while (b[i] % q[j] == 0) b[i] /= q[j], c[i] |= 1 << j;
        if (b[i] % 53 == 0) b[i] = 53;
    }
    up(i, 1, n) up(j, i + 1, n) if (b[i] > b[j]) swap(a[i], a[j]), swap(b[i], b[j]), swap(c[i], c[j]);
    f[0] = 1;
    up(i, 1, n)
    {
        int e                       = ni(b[i]);
        dn(S, 32767, 0) g[S | c[i]] = (g[S | c[i]] + 1ll * (f[S] + g[S]) * a[i]) % p;
        if (b[i] != b[i + 1])
            up(S, 0, 32767) f[S] = (f[S] + (b[i] - 1 ? (b[i] - 1ll) * e % p : 1) * g[S]) % p, g[S] = 0;
    }
    up(S, 0, 32767)
    {
        up(i, 0, 14) if (S >> i & 1) f[S] = f[S] * (q[i] - 1ll) % p * ni(q[i]) % p;
        as                                = (as + f[S]) % p;
    }
    cout << as;
    return 0;
}
```
备注：
只要最终的 `b[i]` 是 $p_x^n$ 即可，即只要不是 $p_1^n \times p_2^m$ 都行，因此53这样特判不影响正确性。
相同的 $p_i$ 其中 $p_i > 53$ 必须同时处理，即对应 `b[i] != b[i + 1]` 的部分，否则重复统计。
```cpp ShowLineNumbers title = "A_0.cpp"
const int N = 2200, q[] = {2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47};

constexpr int P = 998244353;

constexpr i64 power(i64 a, u64 b, i64 res = 1)
{
    for (; b != 0; b /= 2, a *= a, a %= P) {
        if (b & 1) {
            res *= a;
            res %= P;
        }
    }
    return res;
}

void solve()
{
    int n;
    cin >> n;
    vector<int> arr(n + 1);
    vector<int> stat(n + 1);
    for (int i = 1; i <= n; i++) {
        cin >> arr[i];
    }
    vector<int> brr(n + 1);
    for (int i = 1; i <= n; i++) {
        i64 w = arr[i];
        for (int j = 0; j < 15; j++) {
            while (w % q[j] == 0) {
                stat[i] |= 1 << j;
                w /= q[j];
            }
        }
        if (w % 53 == 0) w = 53;
        brr[i] = w;
    }
    for (int i = 1; i <= n; i++)
        for (int j = i + 1; j <= n; j++)
            if (brr[i] > brr[j]) swap(arr[i], arr[j]), swap(brr[i], brr[j]), swap(stat[i], stat[j]);
    constexpr int B = (1 << 15) - 1;
    vector<i64>   ndp(B + 1);
    vector<i64>   cdp(B + 1);
    cdp[0] = 1;
    for (int i = 1; i <= n; i++) {
        for (int j = B; j >= 0; j--) {
            ndp[j | stat[i]] += (cdp[j] + ndp[j]) * arr[i] % P;
            ndp[j | stat[i]] %= P;
        }
        if (i == n || brr[i] != brr[i + 1]) {
            i64 e = (brr[i] - 1 ? ((brr[i] - 1) % P * power(brr[i], P - 2) % P) : 1);
            for (int j = B; j >= 0; j--) {
                cdp[j] += ndp[j] * e % P;
                cdp[j] %= P;
                ndp[j] = 0;
            }
        }
    }
    i64 ans = 0;
    for (int i = 0; i < 15; ++i) {
        i64 e = (q[i] - 1ll) % P * power(q[i], P - 2) % P;
        for (int S = 0; S <= B; ++S) {
            if (S >> i & 1) cdp[S] = cdp[S] * e % P;
        }
    }
    for (int S = 0; S <= B; ++S) {
        ans = (ans + cdp[S]) % P;
    }
    ans += P;
    ans %= P;
    cout << ans << '\n';
}

signed main(signed argc, char** argv)
{
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    int t = 1;
    while (t--) {
        solve();
    }
    return 0;
}
```
