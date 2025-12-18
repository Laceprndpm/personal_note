---
title:
date:
last_clear:
tags:
grasp:
value:
difficulty:
optional_difficulty:
source:
article:
---
# 题意
## 大致题意
给定一棵树，节点带权，将树划分成若干个联通块，每个联通块的值为所有包含的点的最大值减去最小值。要使所有联通块的值的和最大，输出该最大值
# Solution
有以下观察：
1. 最终每个联通快一定是一条链，如果不是，则可以找到联通块中的最小值和最大值，将路径以外的点删去
2. 这条链上一定单调递增/递减，否则断开的答案更优，如 `{1,3,2,4}` ，断开后 `{1,3},{2,4}` 答案更优
那么就可以在树上进行dp
记 
-  `dp[u][0]` 为该点作为一段连续递减路径的头时，其子树中最大的贡献的值
-  `dp[u][1]` 为该点作为一段连续递增路径的头时，其子树中最大的贡献的值
-  `dp[u][2]` 为该点已经不可选时，其子树中最大的贡献的值。(因为这一种情况比前两种都不灵活，或者说前两种都可以是该状态的子集，所以可以令 `dp[u][2]` 为三个状态中最大值)
对于点 `u` 的dp，我们需要统计 `u` 的子树中的三种情况。
- 其作为一个连续递减路径的头，对应 `dp[u][0]`
- 其作为一个连续递增路径的头，对应 `dp[u][1]`
- 其后代间形成了一个路径，对应 `dp[u][2]`
**对于第一种情况**：
我们记所有 `u` 的儿子 `v` 中
`val[v] < val[u]` 为集合 `left_id`
`val[v] = val[u]` 为集合 `mid_id`
`val[v] = val[u]` 为集合 `right_id`
统计所有 `u` 的儿子 `v` 中 `val[v] < val[u]` 的点作为链的第二个元素的贡献是多少，具体的：
$$
\text{left\_val} = (\sum_{i\neq v \land i \in \text{left\_id}}dp[i][2]) + dp[v][0] + (val[u] - val[v])
$$
同时要记录其他的儿子们不参与建链的贡献，直接累加 `dp[i][2]` 即可

$$
\text{mid\_all} = \sum_{i \in \text{mid\_id}}dp[i][2]
$$
$$
\text{right\_all} = \sum_{i \in \text{right\_id}}dp[i][2]
$$
值得注意的是，单独的一个点 `u` 也可以视作一个连续递减段的开头，所以还得记录
$$
\text{left\_all} = \sum_{i \in \text{left\_id}}dp[i][2]
$$
最终令
$$
dp[u][0] = \max(\text{left\_all} , \text{left\_val}) + \text{mid\_all} + \text{right\_all}
$$
**对于第二种情况，与第一种情况一样。**
**对于第三种情况**：
我们可以直接令
$$
dp[u][2] =  \text{left\_val} + \text{right\_val} + \text{mid\_all}
$$
需要注意的是，这三种dp都要和其包含的状态取max，需要谨慎处理。
```cpp showLineNumbers title="D.cpp"
#include <bits/stdc++.h>
using namespace std;
using i64 = long long;

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);
    int n;
    cin >> n;
    vector<i64> arr(n + 1);
    for (int i = 1; i <= n; i++) {
        cin >> arr[i];
    }
    vector<vector<int>> graph(n + 1);
    for (int i = 0; i < n - 1; i++) {
        int u, v;
        cin >> u >> v;
        graph[u].push_back(v);
        graph[v].push_back(u);
    }
    vector<vector<i64>>      dp(n + 1, vector<i64>(3));
    function<void(int, int)> dfs = [&](int u, int fa) -> void {
        for (int v : graph[u]) {
            if (v == fa) continue;
            dfs(v, u);
        }
        vector<int> left_id, right_id, mid_id;
        for (int v : graph[u]) {
            if (v == fa) continue;
            if (arr[v] < arr[u]) {
                left_id.push_back(v);
            } else if (arr[v] > arr[u]) {
                right_id.push_back(v);
            } else {
                mid_id.push_back(v);
            }
        }
        i64 left_all = 0, right_all = 0;
        for (int lit : left_id) {
            left_all += dp[lit][2];
        }
        for (int rit : right_id) {
            right_all += dp[rit][2];
        }
        i64 mid_all = 0;
        for (int mit : mid_id) {
            mid_all += dp[mit][2];
        }
        i64 left_val = 0, right_val = 0;

        for (int lit : left_id) {
            i64 cur  = left_all - dp[lit][2] + dp[lit][0] + (arr[u] - arr[lit]);
            left_val = max(left_val, cur);
        }
        for (int rit : right_id) {
            i64 cur   = right_all - dp[rit][2] + dp[rit][1] + (arr[rit] - arr[u]);
            right_val = max(right_val, cur);
        }
        i64 default_sation = left_all + right_all + mid_all;
        dp[u][0]           = default_sation;
        dp[u][0]           = max(dp[u][0], left_val + right_all + mid_all);
        dp[u][1]           = default_sation;
        dp[u][1]           = max(dp[u][1], left_all + right_val + mid_all);
        dp[u][2]           = left_val + right_val + mid_all;
        dp[u][2]           = max(dp[u][2], dp[u][0]);
        dp[u][2]           = max(dp[u][2], dp[u][1]);
    };
    dfs(1, 1);
    cout << dp[1][2] << '\n';
}
```
