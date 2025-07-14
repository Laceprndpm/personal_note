---
title: E - Knapsack 2
date: 2025-07-06
last_clear: 2025-06-30
tags:
  - tag/dp
grasp: 3
difficulty: 
source: https://atcoder.jp/contests/dp/tasks/dp_e
---
设val为状态就行

# Solution
```c++
    int n, w;
    cin >> n >> w;
    vector<i64> val(1e5 + 50, INF);
    val[0] = 0;
    for (int i = 0; i < n; i++) {
        int weight, value;
        cin >> weight >> value;
        for (int j = 1e5; j >= 0; j--) {
            if (j - value >= 0) val[j] = min(val[j], val[j - value] + weight);
            val[j] = min(val[j], val[j + 1]);
        }
    }
    int ans = upper_bound(all(val), w) - bg(val) - 1;
    cout << ans << '\n';
    return 0;
```
