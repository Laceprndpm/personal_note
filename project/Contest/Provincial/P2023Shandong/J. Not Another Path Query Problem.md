---
title: Not Another Path Query Problem
date: 2025-11-03
last_clear: 2025-11-03
tags:
  - QOJ/Provincial/Shandong/2023
  - SUA
  - tag/细心
  - tag/图论
  - tag/dsu
  - 红温
grasp: 3
value: 5
difficulty:
optional_difficulty: Easy-Medium
source: https://qoj.ac/contest/1280/problem/6679
article:
---
需要注意，当考虑每一层时，约束是否有效，有时候用上层dsu当约束并不优秀
```cpp showLineNumbers
#include <bits/stdc++.h>
using namespace std;
using u64 = unsigned long long;
#define pb push_back
#define bg(x) begin(x)
#define all(x) bg(x), end(x)

/**
 * @brief 并查集
 * 0-index
 */

struct DSU
{
    int component;
    std::vector<int> f, siz;

    DSU() {}

    DSU(int n) : component(n)
    {
        init(n);
    }

    void init(int n)
    {
        f.resize(n);
        std::iota(f.begin(), f.end(), 0);
        siz.assign(n, 1);
    }

    int find(int x)
    {
        while (x != f[x])
        {
            x = f[x] = f[f[x]];
        }
        return x;
    }

    bool same(int x, int y)
    {
        return find(x) == find(y);
    }

    bool merge(int x, int y)
    {
        x = find(x);
        y = find(y);
        if (x == y)
        {
            return false;
        }
        siz[x] += siz[y];
        f[y] = x;
        component--;
        return true;
    }

    int size(int x)
    {
        return siz[find(x)];
    }
};

signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    int n, m, q;
    u64 V;
    cin >> n >> m >> q >> V;
    DSU basic(n + 1);
    vector<tuple<int, int, u64>> edges(m);
    for (int i = 0; i < m; i++)
    {
        int u, v;
        u64 w;
        cin >> u >> v >> w;
        edges[i] = {u, v, w};
        basic.merge(u, v);
    }
    vector<pair<int, int>> querys(q);
    vector<int> ans(q);
    for (int j = 0, u, v; j < q; j++)
    {
        cin >> u >> v;
        querys[j] = {u, v};
    }
    for (int i = 60; i >= 0; i--)
    {
        DSU cur(n + 1);
        for (auto [u, v, w] : edges)
        {
            if (!(w & (1uLL << i))) // 必须用这个约束，因为是边权不是点权
                continue;
            if (basic.same(u, v))
            {
                cur.merge(u, v);
            }
        }
        if (V & (1uLL << i))
        {
            basic = std::move(cur);
        }
        else
        {
            for (int j = 0; j < q; j++)
            {
                auto [u, v] = querys[j];
                if (cur.same(u, v))
                {
                    ans[j] = 1;
                }
            }
        }
    }
    for (int j = 0; j < q; j++)
    {
        auto [u, v] = querys[j];
        if (basic.same(u, v))
        {
            ans[j] = 1;
        }
    }
    for (int i = 0; i < q; i++)
    {
        if (ans[i])
        {
            cout << "Yes\n";
        }
        else
        {
            cout << "No\n";
        }
    }
    return 0;
}
```
#include <bits/stdc++.h>
using namespace std;
using u64 = unsigned long long;
#define pb push_back
#define bg(x) begin(x)
#define all(x) bg(x), end(x)

/**
 * @brief 并查集
 * 0-index
 */

struct DSU
{
    int component;
    std::vector<int> f, siz;

    DSU() {}

    DSU(int n) : component(n)
    {
        init(n);
    }

    void init(int n)
    {
        f.resize(n);
        std::iota(f.begin(), f.end(), 0);
        siz.assign(n, 1);
    }

    int find(int x)
    {
        while (x != f[x])
        {
            x = f[x] = f[f[x]];
        }
        return x;
    }

    bool same(int x, int y)
    {
        return find(x) == find(y);
    }

    bool merge(int x, int y)
    {
        x = find(x);
        y = find(y);
        if (x == y)
        {
            return false;
        }
        siz[x] += siz[y];
        f[y] = x;
        component--;
        return true;
    }

    int size(int x)
    {
        return siz[find(x)];
    }
};

signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    int n, m, q;
    u64 V;
    cin >> n >> m >> q >> V;
    DSU basic(n + 1);
    vector<tuple<int, int, u64>> edges(m);
    for (int i = 0; i < m; i++)
    {
        int u, v;
        u64 w;
        cin >> u >> v >> w;
        edges[i] = {u, v, w};
        basic.merge(u, v);
    }
    vector<pair<int, int>> querys(q);
    vector<int> ans(q);
    for (int j = 0, u, v; j < q; j++)
    {
        cin >> u >> v;
        querys[j] = {u, v};
    }
    for (int i = 60; i >= 0; i--)
    {
        DSU cur(n + 1);
        for (auto [u, v, w] : edges)
        {
            if (!(w & (1uLL << i)))
                continue;
            if (basic.same(u, v))
            {
                cur.merge(u, v);
            }
        }
        if (V & (1uLL << i))
        {
            basic = std::move(cur);
        }
        else
        {
            for (int j = 0; j < q; j++)
            {
                auto [u, v] = querys[j];
                if (cur.same(u, v))
                {
                    ans[j] = 1;
                }
            }
        }
    }
    for (int j = 0; j < q; j++)
    {
        auto [u, v] = querys[j];
        if (basic.same(u, v))
        {
            ans[j] = 1;
        }
    }
    for (int i = 0; i < q; i++)
    {
        if (ans[i])
        {
            cout << "Yes\n";
        }
        else
        {
            cout << "No\n";
        }
    }
    return 0;
}