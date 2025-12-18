---
title: Game on a Tree
date: 2025-09-03
last_clear: 2025-09-03
tags:
  - tag/math/概率
  - tag/计数/容斥
grasp: 2
difficulty: 2100
optional_difficulty: Medium
source: https://ac.nowcoder.com/acm/contest/108303/J
article:
---
# 前言
## 总结
关注处理后的图有什么性质，以及一些概率论的基本性质，以及容斥思想的应用
## 关于容斥
严格地讲不是经典容斥
关键差别：
1. 只有一条链（线性偏序），没有复杂的多集合交集
2. 只需一层减法，不用多项式加减
但确实有容斥思想
# 题意
小 Q 正在游玩一个树上游戏。

游戏的规则如下：每个回合，小 Q 可以选择树上一个未被删除的点，将其以及其所有的直接连边删除。  
当所有边都被删除时，游戏结束（注意，节点不必全部都被删除）。

现在小 Q 想要知道，如果每一回合均匀随机地选取一个可以删除的节点，游戏期望进行多少个回合后结束？  
输出答案对 $998244353$ 取模后的结果。
## Input
- 第一行输入一个正整数 $n$ $(2 \le n \le 5000)$，表示树的节点数。  
- 接下来的 $n-1$ 行，每行包含两个正整数 $x, y$ $(1 \le x, y \le n)$，表示 $x$ 到 $y$ 之间有一条边。
## Output
输出一行一个整数，表示答案。
## 形式化
给定一棵 `n` 个节点的树，每次从未删除的节点中随机选择一个，将其以
及其直接连边删除，直到树中所有边都被删除为止，问期望进行多少次
操作。
# Solution
## std
操作结束时，相邻的两个节点至少有一个需要被删除，否则它们之间的边不会被删除。  
因此剩下的节点必然是树的一个独立集。

若我们能够知道结束时未被删除的点数为 $k$，则 $n - k$ 即为操作的次数。  
因此我们可以尝试枚举最终独立集的大小 $k$，计算 $k$ 个点的独立集方案数为 $f$，  
则将这 $k$ 个点置于最后的排列共有 $(n - k)! k! \times f$ 种，  
这就得到操作次数小于等于 $n - k$ 的方案数，使用容斥即可得到每种操作次数的方案数。

树形 DP，时间复杂度为 $O(n^2)$。
```cpp showLineNumbers title="lace"
void solve()
{
    int n;
    cin >> n;
    vector<vector<int>> graph(n + 1);
    for (int i = 0; i < n - 1; i++) {
        int u, v;
        cin >> u >> v;
        graph[u].eb(v);
        graph[v].eb(u);
    }
    vector<vector<Z>> dp0(n + 1);
    vector<vector<Z>> dp1(n + 1);

    auto conv = [](vector<Z> const &vec1, vector<Z> const &vec2) -> vector<Z> {
        vector<Z> res(vec1.size() + vec2.size() - 1, 0);
        for (int i = 0; i < vec1.size(); i++) {
            for (int j = 0; j < vec2.size(); j++) {
                res[i + j] += vec1[i] * vec2[j];
            }
        }
        return res;
    };
    auto add = [](vector<Z> vec1, vector<Z> vec2) -> vector<Z> {
        if (vec1.size() < vec2.size()) {
            swap(vec1, vec2);
        }
        for (int i = 0; i < vec2.size(); i++) {
            vec1[i] += vec2[i];
        }
        return vec1;
    };
    auto dfs = [&](auto &&self, int u, int fa) -> void {
        for (int v : graph[u]) {
            if (v == fa) continue;
            self(self, v, u);
        }
        vector<Z> cur1{0, 1};
        vector<Z> cur0{1};
        for (int v : graph[u]) {
            if (v == fa) continue;
            cur1 = conv(cur1, dp0[v]);
            cur0 = conv(cur0, add(dp0[v], dp1[v]));
        }
        dp1[u] = std::move(cur1);
        dp0[u] = std::move(cur0);
    };
    dfs(dfs, 1, 1);
    auto      res = add(dp0[1], dp1[1]);  // res[i] 表示 i 个点时独立集的方案数
    vector<Z> tmpans(n + 1);
    for (int i = 0; i < sz(res); i++) {
        tmpans[n - i] = comb.fac(n - i) * comb.fac(i) * res[i];
    }
    for (int i = n; i >= 1; i--) {
        tmpans[i] -= tmpans[i - 1];
    }
    Z ans = 0;
    for (int i = 1; i <= n; i++) {
        ans += i * tmpans[i];
    }
    ans /= comb.fac(n);
    cout << ans << '\n';
}
```