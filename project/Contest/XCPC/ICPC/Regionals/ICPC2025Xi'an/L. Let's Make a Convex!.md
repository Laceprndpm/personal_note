---
title: Let's Make a Convex!
date: 2025-10-22
last_clear: 2025-10-24
tags:
  - QOJ/ICPC/Regionals/Xian/2025
  - tag/二分搜索
  - tag/差分与前缀和
grasp: 4
value: 3
difficulty:
optional_difficulty: Iron
source: https://qoj.ac/contest/2558/problem/14692
article:
---

```cpp showLineNumbers
// 2 3 4 2000 2000 4000
// 考虑当一个边被选择作为最大时，最少选几条其他边
// 从大到小，每次选择一个就直接钦定mn-mx，并且不考虑当前mx > mn的情况
void solve()
{
    int n;
    cin >> n;
    vector<i64> arr(n);
    for (i64 &i : arr) {
        cin >> i;
    }
    sort(all(arr), greater<>());
    vector<i64> prefix(1, 0);
    for (i64 i : arr) {
        prefix.push_back(prefix.back() + i);
    }
    vector<i64> ans(n + 1);
    i64         mn = n + 1;
    for (int i = 0; i < n; i++) {
        i64  curlen = arr[i];
        i64  curval = arr[i] + prefix[i + 1];
        auto it     = upper_bound(all(prefix), curval);
        if (it == prefix.end()) continue;
        i64 choose = it - prefix.begin() - (i + 1);
        choose     = max(choose, 2ll);
        choose++;
        for (int j = choose; j < mn; j++) {
            if (i + j > n) break;
            ans[j] = prefix[i + j] - prefix[i];
        }
        mn = min(choose, mn);
    }
    for (int i = 1; i <= n; i++) {
        cout << ans[i] << ' ';
    }
    cout << '\n';
}
```
