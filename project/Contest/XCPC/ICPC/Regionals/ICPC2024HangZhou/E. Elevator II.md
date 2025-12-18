---
title: Elevator II
date: 2025-10-31
last_clear: 2025-10-31
tags:
  - QOJ/ICPC/Regionals/Hangzhou/2024
  - SUA
  - tag/贪心
grasp: 5
value: 3
difficulty:
optional_difficulty: Easy-Medium
source: https://qoj.ac/contest/1893/problem/9730
article:
---
```cpp
#include <bits/stdc++.h>

#include <algorithm>
#include <numeric>
#include <utility>
#include <vector>
using namespace std;
#define int long long
// vectors
#define sz(x)   int(size(x))
#define bg(x)   begin(x)
#define all(x)  bg(x), end(x)
#define rall(x) rbegin(x), rend(x)
#define sor(x)  sort(all(x))
#define rsz     resize
#define ins     insert
#define pb      push_back
#define eb      emplace_back
#define ft      front()
#define bk      back()
#define mt      make_tuple
#define fi      first
#define se      second
using i64 = long long;

void solve()
{
    int n, f;
    cin >> n >> f;
    vector<pair<i64, i64>> eval(n);
    for (int i = 0; i < n; i++) {
        cin >> eval[i].fi >> eval[i].se;
    }
    vector<int> idx(n);
    iota(all(idx), 0);
    sort(all(idx), [&](int a, int b) { return eval[a].fi < eval[b].fi; });
    i64         ans = 0;
    vector<int> way;
    vector<int> way2;
    for (i64 j = 0; j < n; j++) {
        int i = idx[j];
        if (eval[i].se >= f) {
            way.pb(i);
        } else {
            way2.pb(i);
        }
        ans += max(eval[i].fi - f, 0ll);
        f = max(f, eval[i].se);
        ans += eval[i].se - eval[i].fi;
    }
    sort(all(way2), [&](int a, int b) { return eval[a].se > eval[b].se; });
    cout << ans << '\n';
    way.ins(way.end(), all(way2));
    for (int i : way) {
        cout << i + 1 << ' ';
    }
    cout << '\n';
}

signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);
    int t;
    cin >> t;
    while (t--) {
        solve();
    }
}
```
