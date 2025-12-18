---
title: AUS
date: 2025-10-31
last_clear: 2025-10-31
tags:
  - QOJ/ICPC/Regionals/Hangzhou/2024
  - SUA
  - tag/dsu
grasp: 4
value: 0
difficulty:
optional_difficulty: Easy
source: https://qoj.ac/contest/1893/problem/9726
article:
---
签到
```cpp showLineNumbers
#include <bits/stdc++.h>

#include <set>
#include <utility>
using namespace std;

/**
 * @brief 并查集
 * 0-index
 */

struct DSU {
    int              component;
    std::vector<int> f, siz;

    DSU() {}

    DSU(int n) : component(n) { init(n); }

    void init(int n)
    {
        f.resize(n);
        std::iota(f.begin(), f.end(), 0);
        siz.assign(n, 1);
    }

    int find(int x)
    {
        while (x != f[x]) {
            x = f[x] = f[f[x]];
        }
        return x;
    }

    bool same(int x, int y) { return find(x) == find(y); }

    bool merge(int x, int y)
    {
        x = find(x);
        y = find(y);
        if (x == y) {
            return false;
        }
        siz[x] += siz[y];
        f[y] = x;
        component--;
        return true;
    }

    int size(int x) { return siz[find(x)]; }
};

void solve()
{
    string s1, s2, s3;
    cin >> s1 >> s2 >> s3;
    int n1 = s1.size(), n2 = s2.size(), n3 = s3.size();
    if (n1 != n2) {
        cout << "NO\n";
        return;
    }
    DSU dsu(26);
    for (int i = 0; i < n1; i++) {
        dsu.merge(s1[i] - 'a', s2[i] - 'a');
    }
    if (n1 != n3) {
        cout << "YES\n";
        return;
    }
    for (int i = 0; i < n1; i++) {
        if (!dsu.same(s1[i] - 'a', s3[i] - 'a')) {
            cout << "YES\n";
            return;
        }
    }
    cout << "NO\n";
}

int main()
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
