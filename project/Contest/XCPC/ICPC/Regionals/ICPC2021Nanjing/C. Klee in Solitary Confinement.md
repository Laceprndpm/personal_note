---
title: Klee in Solitary Confinement
date: 2025-10-30
last_clear: 2025-10-30
tags:
  - QOJ/ICPC/Regionals/Nanjing/2021
  - SUA
  - tag/math
grasp: 3
value: 3
difficulty:
optional_difficulty: Easy
source: https://qoj.ac/contest/1236/problem/6433
article: https://zhuanlan.zhihu.com/p/441174109
---
```cpp showLineNumbers
#include <bits/stdc++.h>
using namespace std;
using i64 = long long;
#define pb push_back
#define bg(x) begin(x)
#define all(x) bg(x), end(x)
signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);
    int n, k;
    cin >> n >> k;
    constexpr int padding = 1e6;
    vector<vector<pair<int, int>>> pos(2e6 + 50);
    for (int i = 1; i <= n; i++)
    {
        int v;
        cin >> v;
        v += padding;
        pos[v].pb({i, v});
    }
    int ans = 0;
    vector<pair<int, int>> newvec(n);
    vector<int> idx(2e6 + 1);
    iota(all(idx), 0);
    sort(all(idx), [&](int a, int b)
         { return pos[a].size() > pos[b].size(); });
    for (int i : idx)
    {
        int curans = pos[i].size();
        if (i - k >= 0 && i - k <= 2e6 && k)
        {
            if (pos[i].size() + pos[i - k].size() <= ans)
                continue;
            merge(pos[i].begin(), pos[i].end(), pos[i - k].begin(), pos[i - k].end(), bg(newvec));
            int val = 0;
            for (int j = 0; j < pos[i].size() + pos[i - k].size(); j++)
            {
                int v = newvec[j].second;
                ans = max(ans, curans + val);
                if (v != i)
                {
                    val++;
                }
                else
                {
                    val = max(val - 1ll, 0ll);
                }
            }
            ans = max(ans, curans + val);
        }
        ans = max(ans, curans);
    }
    cout << ans << '\n';
}
```
