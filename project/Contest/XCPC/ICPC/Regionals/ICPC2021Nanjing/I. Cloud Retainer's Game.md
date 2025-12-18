---
title: Cloud Retainer's Game
date: 2025-10-30
last_clear: 2025-10-30
tags:
  - QOJ/ICPC/Regionals/Nanjing/2021
  - SUA
  - tag/dp
  - TOOLESS
grasp: 3
value: 4
difficulty:
optional_difficulty: Easy-Medium
source: https://qoj.ac/contest/1236/problem/6439
article: https://zhuanlan.zhihu.com/p/441174109
---

```cpp showLineNumbers
#include <bits/stdc++.h>
using namespace std;
using i64 = long long;
#define pb push_back
#define bg(x) begin(x)
#define all(x) bg(x), end(x)
#define fi first
#define se second

inline constexpr i64 safeMod(i64 x, i64 m)
{
    x %= m;
    if (x < 0)
    {
        x += m;
    }
    return x;
}
inline void solve()
{
    i64 H;
    cin >> H;
    int n;
    cin >> n;
    vector<array<i64, 3>> act(n);
    for (int i = 0; i < n; i++)
    {
        cin >> act[i][0] >> act[i][1];
        act[i][2] = 0;
    }
    int m;
    cin >> m;
    act.resize(n + m);
    for (int i = 0; i < m; i++)
    {
        cin >> act[i + n][0] >> act[i + n][1];
        act[i + n][2] = 1;
    }
    sort(all(act));
    map<int, int> curup;
    map<int, int> curdown;
    auto mflip = [&H](int k) constexpr -> int
    {
        int newy = 2 * H - k;
        return newy % (2 * H);
    };
    curup[0] = 0;
    curdown[mflip(0)] = 0;
    const int H2 = 2 * H;
    for (int i = 0; i < n + m; i++)
    {
        if (act[i][2] == 1)
        {
            int id1 = safeMod(act[i][0] + act[i][1], H2);
            if (curdown.find(id1) != curdown.end())
            {
                curdown[id1]++;
                curup[mflip(id1)]++;
            }
            int id2 = safeMod(act[i][1] - act[i][0], H2);
            if (curup.find(id2) != curup.end())
            {
                curup[id2]++;
                curdown[mflip(id2)]++;
            }
        }
        else
        {
            int id1 = safeMod(act[i][0] + act[i][1], H2);         // down
            int id2 = safeMod(2 * H + act[i][1] - act[i][0], H2); // up
            int id3 = mflip(id1), id4 = mflip(id2);
            if (curdown.find(id1) != curdown.end() || curup.find(id2) != curup.end())
                curdown[id1] = curup[id3] = curup[id2] = curdown[id4] = max(curdown[id1], curup[id2]);
        }
    }
    cout << max(max_element(all(curdown), [](const auto &a, const auto &b)
                            { return a.se < b.se; })
                    ->se,
                max_element(all(curup), [](const auto &a, const auto &b)
                            { return a.se < b.se; })
                    ->se)
         << '\n';
}
signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);
    int t;
    cin >> t;
    while (t--)
    {
        solve();
    }
}
```