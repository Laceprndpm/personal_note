---
title: Expanding STACKS!
date: 2025-10-20
last_clear: 2025-10-30
tags:
  - QOJ/ICPC/Regionals/Latin_America
  - tag/图论/二分图
  - QOJ/UCUP/2nd/Stage_26_Latin
  - tag/thinking/attention
grasp: 4
value: 5
difficulty:
optional_difficulty: Easy-Medium
source: https://qoj.ac/contest/1576/problem/8509
article:
---
需要注意到，如果in和out的区间发生相交且不是包含关系，则不能同时选择。那么就可以直接枚举所有的顾客关系，然后建一个图，判断是不是二分图即可
```cpp
void solve()
{
    int n;
    cin >> n;
    vector<int> arr(2 * n);
    vector<int> in(n + 1);
    vector<int> out(n + 1);
    for (int i = 0; i < 2 * n; i++) {
        cin >> arr[i];
        if (arr[i] > 0) {
            in[arr[i]] = i;
        } else {
            out[-arr[i]] = i;
        }
    }
    vector<vector<int>> adj(n + 1);
    for (int i = 1; i <= n; i++) {
        int         l = in[i];
        int         r = out[i];
        vector<int> cnt(n + 1);
        for (int j = l + 1; j < r; j++) {
            cnt[abs(arr[j])]++;
        }
        for (int j = 1; j <= n; j++) {
            if (cnt[j] == 1) {
                adj[i].pb(j);
            }
        }
    }
    vector<int> color(n + 1, -1);
    bool        ok  = 1;
    auto        dfs = [&](this auto self, int u, int fa) -> void {
        for (int v : adj[u]) {
            if (fa == v) continue;
            if (color[v] != -1 && color[v] == color[u]) {
                ok = 0;
            }
            if (color[v] == -1) {
                color[v] = !color[u];
                self(v, u);
            }
        }
    };
    for (int i = 1; i <= n; i++) {
        if (color[i] == -1) {
            color[i] = 1;
            dfs(i, i);
        }
    }
    if (!ok) {
        cout << "*";
        return;
    }
    for (int i = 1; i <= n; i++) {
        cout << (color[i] ? 'G' : 'S');
    }
    cout << '\n';
}
```
