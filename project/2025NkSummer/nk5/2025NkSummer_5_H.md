---
title: VI Civilization
date: 2025-10-04
last_clear: 2025-10-04
tags:
  - tag/dp
  - tag/thinking/约束分析
  - tag/二分搜索
grasp: 2
value: 4
difficulty:
optional_difficulty: Easy-Medium
source: https://ac.nowcoder.com/acm/contest/108302/H
article: file:///home/patchouli/Voile/project/2025NkSummer/attachment/2025NkSummer/5/2025_%E7%89%9B%E5%AE%A2%E6%9A%91%E6%9C%9F%E5%A4%9A%E6%A0%A1%E8%AE%AD%E7%BB%83%E8%90%A55%E9%A2%98%E8%A7%A3.pdf
---
注意到生产力和科技值不可分割，且科技值在研发出下一个科技前没有变化，通过该约束可避免记录当前科技研究/生产的状态，从而设计出 `dp[i][j][x]` 为前`i`轮解锁了前`j`个科技，且有`x`轮没有使用生产力的最大科技槽。
```cpp
void solve()
{
    i64 m, s, t;
    cin >> m >> s >> t;
    int n;
    cin >> n;
    vector<array<i64, 4>> arr(n + 1);
    for (int i = 1; i <= n; i++) {
        for (int j = 0; j < 4; j++) {
            cin >> arr[i][j];
        }
    }
    vector<i64> prefix(n + 1);
    for (int i = 1; i <= n; i++) {
        prefix[i] = prefix[i - 1] + arr[i][1];
    }
    auto relax   = [](int& a, int b) { a = max(a, b); };
    auto checker = [&](int p) {
        using Val = int;
        vector<vector<vector<Val>>> dp(t + 1, vector<vector<Val>>(n + 1, vector<Val>(t + 1, -INF)));
        dp[0][0][0] = 0;
        for (int turn = 0; turn < t; turn++) {
            for (int sci = 0; sci <= n; sci++) {
                for (int unused = 0; unused <= turn; unused++) {
                    i64 scipoints = (prefix[sci] + m);
                    relax(dp[turn + 1][sci][unused + 1], dp[turn][sci][unused] + scipoints);
                    if (sci == n) continue;
                    i64 p1_turns = (arr[sci + 1][0] + scipoints - 1) / scipoints;
                    if (p1_turns + turn <= t) {
                        relax(dp[turn + p1_turns][sci + 1][unused + p1_turns], dp[turn][sci][unused]);
                    }
                    if (p == 0) continue;
                    i64 needunused = (arr[sci + 1][2] + p - 1) / p;
                    i64 p2_turns   = (arr[sci + 1][0] - arr[sci + 1][3] + scipoints - 1) / scipoints;
                    if (unused + p2_turns - needunused >= 0 && turn + p2_turns <= t)
                        relax(dp[turn + p2_turns][sci + 1][unused + p2_turns - needunused], dp[turn][sci][unused]);
                }
            }
        }
        for (int sci = 0; sci <= n; sci++) {
            for (int un = 0; un <= t; un++) {
                if (dp[t][sci][un] >= s) {
                    return true;
                }
            }
        }
        return false;
    };
    if (!checker(INF)) {
        cout << -1 << '\n';
        // return;
        return;
    }
    i64 l = 0, r = INF;
    i64 ans = -1;
    while (l <= r) {
        i64 mid = (l + r) / 2;
        if (checker(mid)) {
            r   = mid - 1;
            ans = mid;
        } else {
            l = mid + 1;
        }
    }
    cout << ans << '\n';
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
    int t = 1;
    while (t--) {
        solve();
    }
    return 0;
}
```