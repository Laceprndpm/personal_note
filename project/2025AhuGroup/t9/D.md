---
title: " D. 有限小数"
date: 2025-10-03
last_clear: 2025-10-04
tags:
  - tag/math
  - tag/math/exgcd
grasp: 2
value: 5
difficulty:
optional_difficulty: Silver
source: https://zhuanlan.zhihu.com/p/16058248745
article: https://zhuanlan.zhihu.com/p/16058248745
---

# Solution

## 赛时错解
```cpp ShowLineNumbers
vector<i64>   ca;
constexpr int MAXN = 2e9;

void maininit()
{
    deque<i64> loveme;
    loveme.pb(1);
    while (!loveme.empty()) {
        i64 i = loveme.front();
        loveme.pop_front();
        ca.pb(i);
        if (i * 2 > MAXN) continue;
        loveme.pb(i * 2);
        if (i * 5 > MAXN) continue;
        loveme.pb(i * 5);
    }
    sor(ca);
    ca.erase(unique(all(ca)), ca.end());
}

void solve()
{
    i64 a, b;
    cin >> a >> b;
    i64 ccb = b;
    while (ccb % 2 == 0) {
        ccb /= 2;
    }
    while (ccb % 5 == 0) {
        ccb /= 5;
    }
    if (ccb == 1) {
        cout << 0 << ' ' << 1 << '\n';
        return;
    }
    pair<i64, i64> c_ica = {INF, 0};
    for (i64 ica : ca) {
        // if (b * ica > MAXN) break;
        i64 val  = a * ica;
        i64 meet = ((val - 1) / ccb + 1) * ccb;
        i64 need = meet - val;
        i64 gc   = gcd(need, ica * b);
        i64 c = need / gc, d = ica * b / gc;
        if (d > 1e9) continue;
        if (c < c_ica.fi) {
            c_ica = {c, d};
        }
    }
    // a / b + c / d
    // ad + cb / bd
    // ad = val, cb = need * b
    cout << c_ica.fi << ' ' << c_ica.se << '\n';
}

signed main(signed argc, char** argv)
{
    maininit();
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
```
我注意到了什么？
注意到最终分母和分子约分后的结果一定是
$$
\frac{A}{B} \ \ 其中B=2^a\times 5^b
$$
时，为可约分分数
没注意到什么？
我没注意到
$$
\frac{a}{b} + \frac{c}{d} = \frac{ad + bc}{bd} = \frac{A}{B}
$$
中， $d$ 可以只含有 $w(d脱去2和5的质因子后的)$ 或者说，我不会枚举 $d$ 如果在 $b \nmid d$ 的情况。

但是实际上，可以用 `exgcd` 来找到满足该情况的最小的 $c$。
