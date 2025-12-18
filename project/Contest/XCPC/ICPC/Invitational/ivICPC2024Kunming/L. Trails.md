---
title: Trails
date: 2025-10-31
last_clear: 2025-10-31
tags:
  - QOJ/ICPC/Invitational/Kunming/2024
  - SUA
  - tag/dp/LIS
grasp: 5
value: 2
difficulty:
optional_difficulty: Medium
source: https://qoj.ac/contest/1802/problem/9431
article:
---
```cpp showLineNumbers
void solve()
{
    i64 n, p, q;
    cin >> n >> p >> q;
    vector<pair<int, int>> road(n);
    for (int i = 0; i < n; i++) {
        cin >> road[i].fi >> road[i].se;
        road[i].fi++, road[i].se++;
    }
    sort(all(road), [](pair<int, int> a, pair<int, int> b) {
        if (a.fi == b.fi) return a.se > b.se;
        return a.fi < b.fi;
    });
    vector<i64> dp(n + 1, q + 1);
    i64         ficol = (0 + q) * (q + 1) / 2;
    i64         lacol = (p + p + q) * (q + 1) / 2;
    i64         ans   = (ficol + lacol) * (p + 1) / 2;
    for (auto [x, y] : road) {
        if (x > p || y > q) continue;
        auto it = lower_bound(all(dp), y);
        int  op = *it;
        *it     = y;
        ans -= (op - y) * (p + 1 - x);
    }
    cout << ans << '\n';
}

```
