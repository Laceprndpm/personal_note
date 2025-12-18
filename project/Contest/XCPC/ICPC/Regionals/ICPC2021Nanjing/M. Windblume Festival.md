---
title: Windblume Festival
date: 2025-10-30
last_clear: 2025-10-30
tags:
  - QOJ/ICPC/Regionals/Nanjing/2021
  - SUA
  - tag/细心
grasp: 4
value: 4
difficulty:
optional_difficulty: CheckIn
source: https://qoj.ac/contest/1236/problem/6443
article: https://zhuanlan.zhihu.com/p/441174109
---
注意特判，尤其是有相同元素时
```cpp showLineNumbers
#include <bits/stdc++.h>
using namespace std;
using i64 = long long;
#define pb push_back
#define bg(x) begin(x)
#define all(x) bg(x), end(x)
void solve()
{
    int n;
    cin >> n;
    vector<int> arr(n);
    for (int i = 0; i < n; i++)
    {
        cin >> arr[i];
    }
    if (n == 1)
    {
        cout << arr[0] << '\n';
        return;
    }
    int maxid = max_element(all(arr)) - bg(arr);
    int minid1 = min_element(bg(arr), bg(arr) + maxid) - bg(arr);
    int minid2 = min_element(bg(arr) + maxid + 1, end(arr)) - bg(arr);
    int minid;
    if (minid1 == maxid)
    {
        minid = minid2;
    }
    else if (minid2 == n)
    {
        minid = minid1;
    }
    else
    {
        minid = arr[minid1] < arr[minid2] ? minid1 : minid2;
    }
    i64 ans1 = 0;
    ans1 += arr[maxid];
    ans1 -= arr[minid];
    for (int i = 0; i < n; i++)
    {
        if (i == minid || i == maxid)
            continue;
        ans1 += abs(arr[i]);
    }
    cout << ans1 << '\n';
}
signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);
    int t;
    cin >> t;
    while (t--)
    {
        solve();
    }
}
```
