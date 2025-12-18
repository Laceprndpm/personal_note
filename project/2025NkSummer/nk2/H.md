---
title: Highway Upgrade
date: 2025-09-01
last_clear: 2025-09-01
tags:
  - tag/最短路
  - tag/凸包
grasp: 3
difficulty: 1900
source: https://ac.nowcoder.com/acm/contest/108299/H
article:
---
# 题意
NowLand 的高速公路系统已经使用了几十年，需要进行升级。  

在这个国家，有 $n$ 个城市和 $m$ 条单向高速公路，这些高速公路从一个城市通往另一个城市。一辆车可以使用第 $i$ 条高速公路在 $t_i$ 分钟内从城市 $u_i$ 到达城市 $v_i$。一次关于这条公路的高速公路升级可以将这条路的用时减少 $w_i$ 分钟。每条高速公路都可以进行多次升级。  

作为一个自私的人，NowLand 的总统只考虑自己的利益。升级高速公路系统是他最后的工作，而此后他就退休了。退休后，他将从 NowLand 的首都城市 $1$ 出发前往城市 $n$，在那里度过余生。在这次旅程中，他只会使用高速公路，因此他只关心从城市 $1$ 到城市 $n$ 的时间，并希望尽可能缩短它。  

但是由于政府没有足够的资金，因此高速公路升级的预算是有限的。在预算尚未确定的情况下，总统需要为不同的情况做好准备。于是，他想知道，如果他可以进行 $k$ 次升级，那么他从城市 $1$ 到城市 $n$ 所需的最短时间是多少呢？  

可以保证，使用高速公路是从城市 $1$ 到城市 $n$ 的。
## 形式化
给定一个有向图，有 $n$ 个点和 $m$ 条边，每条边由 $u_i$ 连向 $v_i$，边权为 $t_i$。  
每次操作可以将一条边的权值减小 $w_i$。  
有 $q$ 次查询，每次查询给定 $k$，表示进行 $k$ 次操作，求在此情况下从 $1$ 到 $n$ 的最短路长度。  
可以保证节点 $1$ 能到达节点 $n$。
- $4 \le n \le 10^5$  
- $1 \le m \le 3 \cdot 10^5$  
- $1 \le q \le 3 \cdot 10^5$  
# Solution
## std
假设我们已经找到了一条路径，那么我们应该怎么操作呢？我们一定选择其中 $v_i$ 最大的边进行操作。  
所以，操作的边只能有一条。  
考虑枚举这条边 $(u_i, v_i, t_i, w_i)$，则在走这条边前一定走 $1$ 到 $u_i$ 的最短路，  
走这条边后一定走 $v_i$ 到 $n$ 的最短路。  
于是可以用原图计算 $1$ 到每个点的最短路，用反向边建图可以得到每个点到 $n$ 的最短路。  
设从 $1$ 开始到 $x$ 的最短路长度为 $d_1(x)$，从 $x$ 开始到 $n$ 的最短路长度为 $d_n(x)$， 
则操作 $k$ 次的最短路长度为：
$$
d_1(u_i) + d_n(v_i) + t_i - w_i \cdot k
$$
因此，查询相当于对每一个 $k$ 求上述函数的最小值。因为上面的函数都是关于 $k$ 的一次函数，也就是一系列的直线，  
于是可以维护这些直线形成的可行域（类似于凸包），在分割点序列中进行二分即可。  
时间复杂度为 $O(n + m \log n + q \log m)$。
## my
完全一致，出现多次WA，原因是现找的模板为浮点数二分而不是整数二分。
```c++

template <typename T>
T floor(T a, T b)
{
    return a / b - (a % b && (a ^ b) < 0);
}

template <typename T>
T ceil(T x, T y)
{
    return floor(x + y - 1, y);
}

struct Line {
    i64 a, b;
    i64 xleft;  // 与前一条线交点的横坐标（下凸包的转折点）

    i64 eval(i64 x) const
    {
        return a * x + b;
    }
};

vector<i64> help(vector<pair<i64, i64>> lines, vector<i64> x_query)
{
    sort(lines.begin(), lines.end(), [](const auto &p1, const auto &p2) {
        if (p1.fi == p2.fi) {
            return p1.se < p2.se;
        }
        return p1.fi > p2.fi;
    });

    lines.erase(unique(all(lines)), lines.end());
    vector<Line> hull;

    auto no_D = [](const Line &l1, const Line &l2, const Line &l3) -> bool {
        // a1 * x + b1 = a2 * x + b2
        // (l2.b - l1.b) / (l1.a - l2.a) >= (l3.b - l2.b) / (l2.a - l3.a)
        return ((i128)(l2.b - l1.b)) * (i128)(l2.a - l3.a) >= ((i128)(l3.b - l2.b)) * (i128)(l1.a - l2.a);
    };

    auto xiangxia_jiao = [](const Line &l1, const Line &l2) -> i64 {
        // a1 x + b >= a2 * x + b
        return ceil((l2.b - l1.b), (l1.a - l2.a));
    };

    for (auto [a, b] : lines) {
        Line l = {a, b, -(i64)1e18};
        if (!hull.empty() && l.a == hull.bk.a) continue;

        while (hull.size() >= 2 && no_D(hull[sz(hull) - 2], hull.bk, l)) {
            hull.pop_back();
        }

        if (!hull.empty()) {
            l.xleft = xiangxia_jiao(hull.back(), l);
        }
        hull.push_back(l);
    }

    vector<i64> res;
    for (i64 x : x_query) {
        i64 ans = 0;
        int l = 0, r = (int)hull.size() - 1;
        while (l <= r) {
            int m = (l + r) / 2;
            if (hull[m].xleft <= x) {
                ans = m;
                l   = m + 1;
            } else {
                r = m - 1;
            }
        }
        res.push_back(hull[ans].eval(x));
    }
    return res;
}

void solve()
{
    i64 n, m;
    cin >> n >> m;
    vector<vector<pair<i64, i64>>> graph(n + 1);
    vector<vector<pair<i64, i64>>> r_graph(n + 1);
    vector<pair<i64, i64>>         linear;

    vector<array<i64, 4>> input;
    for (int i = 0; i < m; i++) {
        i64 u, v, ti, wi;
        cin >> u >> v >> ti >> wi;
        graph[u].pb({v, ti});
        r_graph[v].pb({u, ti});
        input.pb({u, v, ti, wi});
    }

    vector<i64> dist1(n + 1, INF);
    vector<i64> dist2(n + 1, INF);

    auto dijstra1 = [&]() {
        vector<int> vis(n + 1);

        priority_queue<pair<i64, i64>, vector<pair<i64, i64>>, greater<>> pq;
        pq.push({0, 1});
        while (!pq.empty()) {
            auto [dist, u] = pq.top();
            pq.pop();

            if (vis[u]) continue;
            vis[u]   = 1;
            dist1[u] = dist;

            for (auto [v, t] : graph[u]) {
                pq.push({t + dist, v});
            }
        }
    };

    auto dijstra2 = [&]() {
        vector<int> vis(n + 1);

        priority_queue<pair<i64, i64>, vector<pair<i64, i64>>, greater<>> pq;
        pq.push({0, n});
        while (!pq.empty()) {
            auto [dist, u] = pq.top();
            pq.pop();
            if (vis[u]) continue;

            vis[u]   = 1;
            dist2[u] = dist;

            for (auto [v, t] : r_graph[u]) {
                pq.push({t + dist, v});
            }
        }
    };

    dijstra1();
    dijstra2();

    for (auto [u, v, ti, wi] : input) {
        linear.eb(-wi, ti + dist1[u] + dist2[v]);
    }

    int q;
    cin >> q;
    vector<i64> query(q);
    for (int i = 0; i < q; i++) {
        cin >> query[i];
    }

    auto res = help(linear, query);
    for (i64 i : res) {
        cout << i << '\n';
    }
}
```