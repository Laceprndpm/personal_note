---
title: Collect the Coins
date: 2025-10-31
last_clear: 2025-10-31
tags:
  - QOJ/ICPC/Invitational/Kunming/2024
  - SUA
  - tag/二分搜索
  - tag/贪心
  - TOOLESS
  - tag/thinking/归纳
grasp: 4
value: 3
difficulty:
optional_difficulty: Medium
source: https://qoj.ac/contest/1802/problem/9427
article: https://qoj.ac/download.php?type=attachments&id=1802&r=2
---
思路很简单，但是赛时几乎没什么人做
武汉邀请赛前写过一次，写了一坨，现在优雅不少
```cpp showLineNumbers
#include <algorithm>
#include <iostream>
#include <utility>
#include <vector>
using namespace std;
using ll  = long long;
using i64 = long long;
#define fi first
#define se second

#define int long long

void solve()
{
    int n;
    cin >> n;
    vector<int> ti(n);
    vector<int> pos(n);
    for (int i = 0; i < n; i++) {
        cin >> ti[i] >> pos[i];
    }

    auto isin = [](int pos, pair<int, int> rg) {
        if ((rg.se >= pos) && (rg.fi <= pos))
            return true;
        else
            return false;
    };
    auto bin = [](pair<int, int> a, pair<int, int> b) -> pair<int, int> {
        int l = min(a.fi, b.fi), r = max(a.se, b.se);
        return {l, r};
    };
    auto check = [&](i64 v) {
        int            last = 0;
        pair<i64, i64> a    = {1, 1e9}, b{1, 1e9};
        for (int i = 0, j = 0; i < n; i++) {
            a.fi -= (ti[i] - last) * v;
            b.fi -= (ti[i] - last) * v;
            a.fi = max(a.fi, 1ll);
            b.fi = max(b.fi, 1ll);
            a.se += (ti[i] - last) * v;
            b.se += (ti[i] - last) * v;
            a.se = min(a.se, ll(1e9));
            b.se = min(b.se, ll(1e9));
            last = ti[i];
            while (j < n && ti[j] <= ti[i]) {
                if (isin(pos[j], a) && isin(pos[j], b)) {
                    a = bin(a, b);
                    b = {pos[j], pos[j]};
                } else if (isin(pos[j], a)) {
                    a = {pos[j], pos[j]};
                } else if (isin(pos[j], b)) {
                    b = {pos[j], pos[j]};
                } else {
                    return false;
                }
                j++;
            }
        }
        return true;
    };
    if (!check(1e9)) {
        cout << -1 << '\n';
        return;
    }
    int ans = 0;
    if (check(0)) {
        cout << 0 << '\n';
        return;
    }
    for (int i = 29; i >= 0; i--) {
        if (!check((1 << i) + ans)) {
            ans += 1 << i;
        }
    }
    cout << ans + 1 << '\n';
}

signed main(signed argc, char** argv)
{
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    int t;
    cin >> t;
    while (t--) {
        solve();
    }
    return 0;
}
```
