---
title: 线段染色
date: 2025-07-25
last_clear: 2025-07-25
tags:
  - tag/dp
  - tag/模拟
grasp: 2
difficulty: 1800
optional_difficulty: Easy-Medium
source: https://acm.hdu.edu.cn/contest/problem?cid=1174&pid=1009
article:
---
[[dp_w]] 与之高度类似，尤其是优化掉第二维的时候
本题写时，dp的状态没有完全想清楚，导致第一个dp只要换个视角就可以写对，但是没有
于是换了dp方式，大量时间浪费
以及前面的线段处理出了岔子，因此打上 `implementation` 的tag
### 赛时O(nlog(n))
因为如果一个线段完全包含了一个线段，那么内部的线段作为一个必要条件就可以代替外部的线段。因此我们可以线段预处理，之后我们发现，其左端点序等于其右端点序（后称线段序）。每个点负责的线段序区间有单调性，那么就可以对每个点进行一次状态转移。设 `dp[i]` 为处理完了前 `i` 个线段的概率，到达点 `j` 时，假设其在线段 `l` 到 `r` 上，那么它可以使
$$
dp[r] = \sum_{l-1}^{r-1} \cdot poss_j
$$
同时
$$
dp[i] = dp[i] \cdot (1 - poss_j) , l - 1\le i \le r-1
$$
因为只有不选才能保持他们本身，之后用线段树维护这个dp过程，就可以得到 $O(n\log(n))$ 的解，只需要预处理10的逆元就可以避免TLE。
```cpp showLineNumbers title="p09.cpp"
{
    int n, m;
    cin >> n >> m;
    vector<int> ai(n + 1);
    for (int i = 1; i <= n; i++) {
        cin >> ai[i];
    }
    vector<pair<int, int>> line(m + 1);
    for (int i = 1; i <= m; i++) {
        int l, r;
        cin >> l >> r;
        line[i] = {l, r};
    }
    sort(all(line), [](auto a, auto b) {
        if (a.fi == b.fi) return a.se > b.se;
        return a.fi < b.fi;
    });
    vector<pair<int, int>> new_line;
    new_line.reserve(m);
    new_line.pb({0, 0});
    for (int i = 1; i <= m; i++) {
        while (new_line.size() > 1 && new_line.bk.se >= line[i].se) {
            new_line.pop_back();
        }
        new_line.pb(line[i]);
    }
    // 预处理线段
    line = new_line;
    m = sz(line) - 1;
    vector<int> prefix_line1(n + 2);
    vector<int> prefix_line2(n + 2);
    for (int i = 1; i <= m; i++) {
        prefix_line1[line[i].se + 1]++;
        prefix_line2[line[i].fi]++;
    }
    for (int i = 1; i <= n; i++) {
        prefix_line1[i] += prefix_line1[i - 1];
        prefix_line2[i] += prefix_line2[i - 1];
    }
    // 预处理点负责区间
    LazySegmentTree<Info, Tag> seg(m + 1);

    seg.modify(0, Info{Z{1}});
    for (int i = 1; i <= n; i++) {
        int l = prefix_line1[i] + 1;
        int r = prefix_line2[i];
        if (l > r) continue;
        auto res1 = seg.rangeQuery(max(l - 1, 0), r);
        res1.x    = res1.x * Z(ai[i]) * Z(700000005);  // 加到r头上
        seg.rangeApply(max(l - 1, 0), r, Tag(1 - Z(ai[i]) * Z(700000005)));
        seg.modify2(r, res1);
    }
    cout << seg.rangeQuery(m, m + 1).x << '\n';
}
	// dp过程
```
### 赛后O(n)
1009的dp可以设 `dp[i][j]` 为考虑前i位时，最后一个1在 `j` 位置时合法的概率，然后把第一维优化掉就行，然后用前缀和来实现O(n)转移，类似于[[dp_w]]
```cpp showLineNumbers title="p09_jiangly.cpp"
void solve()
{
    int n, m;
    std::cin >> n >> m;

    std::vector<int> a(n + 2), b(n + 2);
    for (int i = 1; i <= n; i++) {
        std::cin >> a[i];
        a[i] = i64(a[i]) * inv10 % P;
        b[i] = i64(a[i]) * power(1 - a[i], P - 2) % P;
    }
    b[n + 1] = 1;

    int mul = 1;
    for (int i = 1; i <= n; i++) {
        if (a[i] != 1) {
            mul = i64(mul) * (1 - a[i]) % P;
        }
    }

    std::vector<int> cnt(n + 1);
    for (int i = 1; i <= n; i++) {
        cnt[i] = cnt[i - 1] + (a[i] == 1);
    }

    std::vector<int> f(n + 1);

    for (int i = 0; i < m; i++) {
        int l, r;
        std::cin >> l >> r;
        if (cnt[r] - cnt[l - 1] == 0) {
            f[r] = std::max(f[r], l);
        }
    }

    for (int i = 1; i <= n; i++) {
        f[i] = std::max(f[i], f[i - 1]);  // 最远从哪里转移
    }

    // dp[i]为合法，且最后一个染色的点为i
    std::vector<int> dp(n + 2), pre(n + 2);  // dp[i] = [1, i]点全部合法
    dp[0]  = 1;
    pre[0] = 1;
    for (int i = 1; i <= n + 1; i++) {  // i-1 -> i
        dp[i]  = i64(pre[i - 1] - (f[i - 1] ? pre[f[i - 1] - 1] : 0)) * b[i] % P;
        pre[i] = (pre[i - 1] + dp[i]) % P;
    }

    int ans = i64(dp[n + 1]) * mul % P;
    if (ans < 0) {
        ans += P;
    }
    std::cout << ans << "\n";
}
```
