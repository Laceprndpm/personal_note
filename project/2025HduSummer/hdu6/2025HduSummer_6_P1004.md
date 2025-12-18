---
title: 传送排序
date: 2025-09-08
last_clear:
tags:
  - tag/dp
  - tag/贪心
grasp: 3
difficulty:
optional_difficulty: Easy-Medium
source:
article:
---
# Solution
## my
==WARNING：由于该题解为赛后对着赛时的代码逆向出来，且赛时记忆已经模糊，因此不保证准确==
首先，他最多的操作次数为 $n+1$ 次，即放一个传送门后，暴力传送所有猫猫。
玩几个样例可以发现，连续的上升序列可以减少传送的次数，即有部分猫猫可以不动。而如果不连续，出现了间断，如 $1\ 2\ 4\ 5\ 6\ 3$ 也不会让答案更劣，且后续的连续段也可以产生贡献，就可以由这两个转移写出dp式。
`dp[i]` 表示以 `i` 结尾的未传送段可以少操作 `dp[i]` 次
$$
\max(\text{dp}[val], \text{dp}[val - 1]) \rightarrow \text{dp}[val]
$$
$$
\max(\mathrm{range\_max}(\text{dp}[1..(val - 1)]), \text{dp}[val] )\rightarrow dp[val]
$$
```cpp showLineNumbers title="P1004.cpp"
void solve()
{
    int n;
    cin >> n;
    vector<int> arr(n + 2);
    arr[0] = 0, arr[n + 1] = n + 1;
    for (int i = 1; i <= n; i++) {
        cin >> arr[i];
    }
    Fenwick<Info> fen(n + 5);
    vector<int>   max_n(n + 2, -INF);
    for (int i = 0; i < n + 2; i++) {
        int val     = arr[i];
        int cur_val = 0;
        if (val) {
            cur_val = max(cur_val, fen.sum(val - 1).x);
            cur_val = max(cur_val, max_n[val - 1] + 1);
        }
        fen.add(val, {cur_val});
        max_n[val] = cur_val;
    }
    cout << n + 1 - fen.sum(n + 2).x << '\n';
}
```