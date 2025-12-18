---
title: Perfect Matching
date: 2025-11-06
last_clear: 2025-11-06
tags:
  - QOJ/ICPC/Regionals/Nanjing/2022
  - SUA
  - tag/构造
  - tag/图论/二分图
  - tag/贪心
grasp: 2
value: 5
difficulty:
optional_difficulty: Hard-Medium
source: https://qoj.ac/contest/1093/problem/5423
article:
---
第一次见到二分图匹配，将匹配的对象放在边上而不是点上的，仔细一想应该是因为无法分别左右，所以不能点分二分图。
```cpp showLineNumbers
void solve()
{
    // ai - i = aj - j
    // ai + i = ak + k
    // ak + k = ai + i = ai - i = aj + j
    // 不存在
    // 所以两个集合一定只有一个交点i
    //
    int n;
    cin >> n;
    unordered_map<int, int> mpL, mpR;
    vector<int>             arr(n + 1);
    for (int i = 1; i <= n; i++) {
        cin >> arr[i];
        mpL[arr[i] - i] = mpR[arr[i] + i] = 0;
    }
    int L = 0, R = 0;
    for (auto it = mpL.begin(); it != mpL.end(); it++) it->second = ++L;
    for (auto it = mpR.begin(); it != mpR.end(); it++) it->second = ++R;
    int                            m = L + R;
    vector<vector<pair<int, int>>> adj(m + 1);
    for (int i = 1; i <= n; i++) {
        int x = mpL[arr[i] - i], y = L + mpR[arr[i] + i];
        adj[x].pb({y, i});
        adj[y].pb({x, i});
    }
    vector<int> vis(m + 1);
    auto        bfs = [&](int x) -> int {
        deque<int> que;
        int        cursz = 0;
        que.pb(x);
        while (!que.empty()) {
            int u = que.front();
            que.pop_front();
            for (auto [v, id] : adj[u]) {
                cursz++;
                if (!vis[v]) {
                    vis[v] = 1;
                    que.pb(v);
                }
            }
        }
        return cursz / 2;
    };
    bool                   ok = 1;
    vector<int>            dep(m + 1);
    vector<pair<int, int>> ans;
    auto                   dfs = [&](auto&& self, int u, int fa, int d = 1) -> int {
        dep[u] = d;
        vector<int> vec;
        int         faid;
        for (auto [v, id] : adj[u]) {
            if (v == fa) {
                faid = id;
                continue;
            }
            if (dep[v] > 0) {
                if (dep[v] < dep[u]) vec.push_back(id);
            } else {
                int t = self(self, v, u, d + 1);
                if (t > 0) vec.push_back(t);
            }
        }
        while (vec.size() > 1) {
            int x = vec.back();
            vec.pop_back();
            int y = vec.back();
            vec.pop_back();
            ans.push_back({x, y});
        }
        if (vec.size() == 1) {
            // 还剩一条边没匹配，把连向父节点的边也用来匹配
            assert(fa > 0);
            ans.push_back({vec.back(), faid});
            return -1;
        } else {
            // 连向父节点的边交给父节点匹配
            return faid;
        }
    };
    for (int i = 1; i <= m; i++) {
        if (!vis[i]) {
            vis[i]  = 1;
            int cnt = bfs(i);
            if (cnt % 2 != 0) {
                ok = 0;
                break;
            }
            dfs(dfs, i, 0);
        }
    }
    if (ok) {
        cout << "Yes\n";
        for (auto [u, v] : ans) {
            cout << u << ' ' << v << '\n';
        }
    } else {
        cout << "No\n";
    }
}
```
