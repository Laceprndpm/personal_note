---
title: Crystalfly
date: 2025-10-30
last_clear: 2025-10-30
tags:
  - QOJ/ICPC/Regionals/Nanjing/2021
  - SUA
  - tag/dp/树上dp
grasp: 4
value: 2
difficulty:
optional_difficulty: Easy-Medium
source: https://qoj.ac/contest/1236/problem/6438
article: https://zhuanlan.zhihu.com/p/441174109
---
```cpp showLineNumbers
#include <bits/stdc++.h>
using namespace std;
using i64 = long long;
#define pb push_back
#define bg(x) begin(x)
#define all(x) bg(x), end(x)
#define int long long
void solve()
{
    int n;
    cin >> n;
    vector<int> arr(n + 1), trr(n + 1);
    arr[0] = 0;
    for (int i = 1; i <= n; i++)
        cin >> arr[i];
    for (int i = 1; i <= n; i++)
        cin >> trr[i];
    vector<vector<int>> adj(n + 1);
    vector<array<i64, 2>> dp(n + 1, {0, 0});
    for (int i = 0; i < n - 1; i++)
    {
        int u, v;
        cin >> u >> v;
        adj[u].pb(v);
        adj[v].pb(u);
    }
    auto dfs = [&](auto &&self, int u, int fa) -> void
    {
        vector<int> sons;
        vector<int> t3sons;
        for (int v : adj[u])
        {
            if (v == fa)
                continue;
            self(self, v, u);
            sons.pb(v);
            if (trr[v] >= 3)
            {
                t3sons.pb(v);
            }
        }
        // dp[u][1]为来到u后立即又回去了
        for (int v : sons)
        {
            dp[u][1] += dp[v][0];
        }

        t3sons.pb(0); // 防止空访问
        t3sons.pb(0); // 防止空访问

        sort(bg(t3sons), t3sons.end(), [&](int a, int b)
             { return arr[a] > arr[b]; }); // 找到val最大的两个，且t >= 3的
        if (sons.empty())
            return;
        // dp[u][0]为来到u后就不回去了
        // 先用dp[v][0] + max(arr[v])初始化
        dp[u][0] = dp[u][1] + arr[*max_element(all(sons), [&](int a, int b)
                                               { return arr[a] < arr[b]; })];
        // 枚举去哪个儿子后，又回到了u,再去另一个儿子
        // dp[x][1] + arr[x] + dp[ot..][0] + max(arr[ot..]其中t_ot >= 3)
        for (int x : sons)
        {
            i64 cur1 = dp[x][1] + arr[x] - dp[x][0];
            if (x == t3sons[0])
            {
                cur1 += arr[t3sons[1]];
            }
            else
            {
                cur1 += arr[t3sons[0]];
            }
            cur1 += dp[u][1];
            dp[u][0] = max(dp[u][0], cur1);
        }
    };
    dfs(dfs, 1, 0);
    cout << dp[1][0] + arr[1] << '\n';
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
