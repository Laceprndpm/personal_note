---
title: Binary Tree
date: 2025-10-30
last_clear: 2025-10-30
tags:
  - QOJ/ICPC/Regionals/Nanjing/2024
  - SUA
  - tag/图论/树上问题/重心
  - tag/图论/树上问题/点分治
  - tag/交互
grasp: 2
value: 5
difficulty:
optional_difficulty: Easy-Medium
source: https://qoj.ac/contest/1828/problem/9570
article:
---
注意到树的重心性质，所有儿子的大小都小于树大小除二下取整，即 $\forall v, siz[v] \le \lfloor\frac{n}{2}\rfloor$ ，而二叉树的度数 $deg[u] \le 3$ ，因此
1. $deg[u] = 0$ : 返回自身
2. $deg[u] = 1$ : 询问自己和邻居，返回一个
3. $deg[u] = 2$ : 询问两个儿子，返回的子树 $siz[v] \le \lfloor\frac{n}{2}\rfloor$ 
4. $deg[u] = 3$ : 询问两个==重==儿子，如果 $res = 0, 2$ ，返回的子树$siz[v] \le \lfloor\frac{n}{2}\rfloor$ 。如果 $res = 1$ 返回的轻儿子+自己 $siz[v] + 1 \le \lfloor\frac{n}{2}\rfloor$
如果不询问重儿子，将导致 **WA**
需要注意不能直接抄点分治板子，点分治板子传入的不是真实树大小，无法找到重心（这个平时有复杂度证明，但是这题不太行）
```cpp showLineNumbers
#include <bits/stdc++.h>
using namespace std;
using i64 = long long;
#define fi     first
#define se     second
#define pb     push_back
#define bg(x)  begin(x)
#define all(x) bg(x), end(x)
#define bk     back()

void solve()
{
    int n;
    cin >> n;
    int                 root = 0, MaxTree = 1e9;  // 分别代表重心下标、最大子树大小
    vector<vector<int>> adj(n + 1);

    for (int i = 1; i <= n; i++) {
        int l, r;
        cin >> l >> r;
        if (l) {
            adj[i].pb(l);
            adj[l].pb(i);
        }
        if (r) {
            adj[i].pb(r);
            adj[r].pb(i);
        }
    }
    vector<int> vis(n + 1), siz(n + 1);
    auto        get = [&](auto self, int x, int fa, int n) -> void {  // 获取树的重心
        siz[x]  = 1;
        int val = 0;
        for (auto y : adj[x]) {
            if (y == fa || vis[y]) continue;
            self(self, y, x, n);
            siz[x] += siz[y];
            val = max(val, siz[y]);
        }
        val = max(val, n - siz[x]);
        if (val < MaxTree) {
            MaxTree = val;
            root    = x;
        }
    };
    auto query = [&](int u, int v, int ot = 0) -> tuple<int, int, int> {
        cout << "? " << u << ' ' << v << endl;
        int res;
        cin >> res;
        if (res == 0) {
            return {u, v, ot};
        } else if (res == 1) {
            return {ot, u, v};
        } else {
            return {v, u, ot};
        }
    };
    auto dfz = [&](auto self, int x, int fa) -> int {  // 点分治
        // 标记已经被更新过的旧重心，确保只对子树分治
        if (adj[x].size() == 0) {
            return x;
        }
        if (adj[x].size() == 1) {
            auto [y, a1, a2] = query(adj[x][0], x);
            return y;
        } else if (adj[x].size() == 2) {
            auto [y, a1, a2] = query(adj[x][0], adj[x][1], x);
            if (a1) adj[x].erase(find(all(adj[x]), a1));
            if (a2) adj[x].erase(find(all(adj[x]), a2));
            vis[a1] = vis[a2] = 1;
            MaxTree           = 1e9;
            get(get, y, 0, siz[y]);
            return self(self, root, 0);
        } else {
            auto [y, a1, a2] = query(adj[x][0], adj[x][1], x);
            if (a1) adj[x].erase(find(all(adj[x]), a1));
            if (a2) adj[x].erase(find(all(adj[x]), a2));
            vis[a1] = vis[a2] = 1;
            MaxTree           = 1e9;
            get(get, y, 0, siz[y]);
            return self(self, root, 0);
        }
    };

    get(get, 1, 0, n);
    int ans = dfz(dfz, root, 0);
    cout << "! " << ans << endl;
}

int main()
{

    int t;
    cin >> t;
    while (t--) {
        solve();
    }
}
```