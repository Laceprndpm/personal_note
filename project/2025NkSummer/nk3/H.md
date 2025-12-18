---
title: H - Head out to the Target
date: 2025-07-22
last_clear: 2025-07-22
tags:
  - tag/图论/树上问题/树上倍增
grasp: 3
difficulty: 1700
source: https://ac.nowcoder.com/acm/contest/108300/H
article:
---
# 题意
给定一棵有 $n$ 个结点的树。一枚棋子初始位于 $1$ 号结点。

按序给定 $k$ 个时间段。第 $i$ 个时间段 $[l_i, r_i]$ 中，树上会出现一个目标 $u_i$。保证各时间段两两不交。

在每一时刻，若目标存在、棋子与目标位置不同且连通，则棋子向目标移动一步；否则棋子不动。若此时二者位置相同，则称该时刻二者重合。

你可以在任意时刻切断树上任意数量的边。每条边被切断后不会恢复。

判断棋子能否与目标重合；若能，则最小化重合发生的时刻。

约束：

- $1 \le n \le 10^6$
- $1 \le k \le 10^6$
- $1 \le u_i \le n$
- $1 \le l_i \le r_i \le 10$
# Solution
## my
写了错解，用dfsn序+线段树
正解为树上倍增
```c++
void solve()
{
    int n, k;
    cin >> n >> k;
    vector<int>         fa(n + 1);
    vector<vector<int>> child(n + 1);
    for (int i = 2; i <= n; i++) {
        cin >> fa[i];
        child[fa[i]].eb(i);
    }
    vector<Node> node(k);
    for (int i = 0; i < k; i++) {
        cin >> node[i].u >> node[i].l >> node[i].r;
    }
    vector<int>              in(n + 1);
    vector<int>              out(n + 1);
    int                      cur = -1;
    function<void(int, int)> dfs = [&](int u, int fa) -> void {
        in[u] = ++cur;
        for (int v : child[u]) {
            dfs(v, u);
        }
        out[u] = cur;
    };

    dfs(1, 1);
    vector<Info>                   seg_tmp(n);
    vector<vector<pair<int, int>>> dfsn_list(n);
    for (int i = 0; i < k; i++) {
        int dfsn = in[node[i].u];
        int l = node[i].l, r = node[i].r;
        dfsn_list[dfsn].pb({l, r});
    }

    for (int i = 0; i < n; i++) {
        if (dfsn_list[i].empty()) continue;
        seg_tmp[i] = {i, dfsn_list[i][0].fi, 0};
    }

    SegmentTree<Info> seg(seg_tmp);
    i64               ans = INF;

    function<void(int, int)> dfs2 = [&](int u, int cur_time) -> void {
        auto res = seg.rangeQuery(in[u], in[u] + 1);  // 用于特判root
        if (res.l != INF) {
            ans = min(ans, res.l);
        }
        for (int v : child[u]) {
            int  dfs_l = in[v], dfs_r = out[v] + 1;  // 作弊游客
            auto res            = seg.rangeQuery(dfs_l, dfs_r);
            auto [dfsn, l, ptr] = res;

            if (l == INF) {
                continue;
            }

            ans = min(ans, seg.rangeQuery(in[v], in[v] + 1).l);

            if (l == dfsn_list[dfsn][ptr].se) {
                if (ptr + 1 == sz(dfsn_list[dfsn])) {
                    seg.modify(res.x, {res.x, INF, INF});
                } else {
                    ptr++;
                    seg.modify(dfsn, {dfsn, dfsn_list[dfsn][ptr].fi, ptr});
                }
            } else {
                seg.modify(res.x, {res.x, l + 1, ptr});
            }

            dfs2(v, l + 1);
        }
    };
    dfs2(1, 0);
    cout << (ans == INF ? -1 : ans) << '\n';
}
```
## std
观察到任意时刻可达的点是一个包含根的极大连通块。
考虑直接维护这个连通块，当目标在 $[r_i, l_i]$ 时刻内出现在 $u_i$ 时，相当于
将这个连通块向 $u_i$ 方向扩张 $(r_i − l_i + 1)$ 步。
考虑使用倍增，从 ui 开始向上找到最接近的可达点，然后暴力向下逐
步标记为可达，若在 $(r_i − l_i + 1)$ 步前 $u_i$ 已经被标记可达即可输出答案。
