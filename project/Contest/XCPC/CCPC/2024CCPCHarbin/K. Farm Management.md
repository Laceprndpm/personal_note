---
title: Farm Management
date: 2025-10-20
last_clear: 2025-10-20
tags:
  - QOJ/UCUP/3rd/Stage_14_Harbin
grasp: 4
value: 4
difficulty:
optional_difficulty: Easy-Medium
source: https://qoj.ac/contest/1817/problem/9529
article: https://qoj.ac/download.php?type=attachments&id=1817&r=2
---
```cpp showLineNumbers
void solve()
{
    i64 n, m;
    cin >> n >> m;
    vector<array<i64, 3>> crops(n);
    i128                  minval = 0;
    for (int i = 0; i < n; i++) {
        i64 w, l, r;
        cin >> w >> l >> r;
        crops[i] = {w, l, r};
        minval += l * w;
        m -= l;
    }
    sort(all(crops), [&](auto a, auto b) { return a[0] > b[0]; });
    vector<i64> prefix_time(n + 1);
    vector<i64> prefix_val(n + 1);
    for (int i = 0; i < n; i++) {
        prefix_time[i + 1] = prefix_time[i] + (crops[i][2] - crops[i][1]);
        prefix_val[i + 1]  = prefix_val[i] + (crops[i][2] - crops[i][1]) * crops[i][0];
    }
    i64 ans = 0;
    for (int i = 0; i < n; i++) {
        i64  savetime = m + crops[i][1];
        i64  curval   = minval - crops[i][0] * crops[i][1];
        auto it       = upper_bound(prefix_time.begin(), prefix_time.begin() + i + 1, savetime);  // *it > savetime
        savetime -= prefix_time[it - prefix_time.begin() - 1];
        curval += prefix_val[it - prefix_time.begin() - 1];
        if (savetime) {
            int cur = it - prefix_time.begin() - 1;
            cur     = min(cur, i);
            curval += savetime * crops[cur][0];
        }
        ans = max(ans, curval);
    }
    cout << ans << '\n';
}
```
注意到当选择 `i` 作为节点时，应该在保证所有其他最小值被取到的情况下，尽量取前缀大的。
二分即可