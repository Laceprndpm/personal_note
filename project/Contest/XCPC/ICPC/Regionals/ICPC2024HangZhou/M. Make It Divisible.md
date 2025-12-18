---
title: Make It Divisible
date: 2025-10-31
last_clear: 2025-10-31
tags:
  - QOJ/ICPC/Regionals/Hangzhou/2024
  - SUA
  - tag/数据结构/树/CartesianTree
  - tag/math/gcd
grasp: 5
value: 5
difficulty:
optional_difficulty: Easy-Medium
source: https://qoj.ac/contest/1893/problem/9738
article:
---
笛卡尔树梦开始的地方
```cpp showLineNumbers
#include <bits/stdc++.h>

#include <array>
#include <numeric>
#include <utility>
#include <vector>
using namespace std;
using i64 = long long;
#define int long long

/**
 *@brief 笛卡尔树
 */

template <class T>
struct CartesianTree {
    int                        n;
    vector<int>                stk;
    vector<std::array<int, 2>> child;
    vector<T>                  data;
    int                        treeRoot;
    vector<T>                  rangegcd;

public:
    // 1-idx
    CartesianTree(vector<T>& _init) : n(_init.size() - 1)
    {
        data = vector<T>(_init);
        rangegcd.resize(n + 1);
        child.resize(n + 1);
        init();
        init2();
    }

    void init() noexcept
    {
        stk.resize(n + 1);
        // stk 维护笛卡尔树中节点对应到序列中的下标
        int top = 0;
        for (int i = 1; i <= n; i++) {
            int k = top;                                  // top 表示操作前的栈顶，k 表示当前栈顶
            while (k > 0 && data[stk[k]] > data[i]) k--;  // 维护右链上的节点
            if (k) child[stk[k]][1] = i;                  // 栈顶元素.右儿子 := 当前元素
            if (k < top) child[i][0] = stk[k + 1];        // 当前元素.左儿子 := 上一个被弹出的元素
            stk[++k] = i;                                 // 当前元素入栈
            top      = k;
        }
        treeRoot = stk[1];
    }

    void init2()
    {
        auto dfs = [&](auto&& self, int u) -> void {
            if (child[u][0]) {
                self(self, child[u][0]);
                rangegcd[u] = gcd<i64, i64>(rangegcd[u], abs(data[u - 1] - data[u]));
                rangegcd[u] = gcd<i64, i64>(rangegcd[u], rangegcd[child[u][0]]);
            }
            if (child[u][1]) {
                self(self, child[u][1]);
                rangegcd[u] = gcd<i64, i64>(rangegcd[u], abs(data[u + 1] - data[u]));
                rangegcd[u] = gcd<i64, i64>(rangegcd[u], rangegcd[child[u][1]]);
            }
        };
        dfs(dfs, treeRoot);
    }

    bool check(i64 x)
    {
        for (int i = 1; i <= n; i++) {
            if (rangegcd[i] % (data[i] + x) != 0) {
                return false;
            }
        }
        return true;
    }
};

void solve()
{
    i64 n, k;
    cin >> n >> k;
    vector<int> arr(n + 1);
    for (int i = 1; i <= n; i++) {
        cin >> arr[i];
    }
    CartesianTree<int> ctree(arr);
    int                val = ctree.rangegcd[ctree.treeRoot];
    if (val == 0) {
        i64 ans = (1 + k) * k / 2;
        cout << k << ' ' << ans << '\n';
        return;
    }
    i64 ans = 0;
    int num = 0;
    for (i64 v = 1; v * v <= val; v++) {
        if (val % v == 0) {
            i64 x = v - ctree.data[ctree.treeRoot];
            if (x > 0 && x <= k && ctree.check(x)) {
                num++;
                ans += x;
            }
            i64 v2 = val / v;
            x      = v2 - ctree.data[ctree.treeRoot];
            if (v * v != val && x > 0 && x <= k && ctree.check(x)) {
                num++;
                ans += x;
            }
        }
    }
    cout << num << ' ' << ans << '\n';
}

signed main()
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
