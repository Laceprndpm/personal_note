---
title: Be Positive
date: 2025-10-31
last_clear: 2025-10-31
tags:
  - QOJ/ICPC/Invitational/Kunming
  - SUA
  - tag/贪心
grasp: 5
value: 1
difficulty:
optional_difficulty: Easy
source: https://qoj.ac/contest/1802/problem/9428
article:
---
最小字典序，那就直接贪心吧
```cpp showLineNumbers
void solve()
{
    int n;
    cin >> n;
    // 第一个必须1
    // 1023 0123
    if (n % 4 == 0 || n == 1) {
        cout << "impossible\n";
        return;
    }
    vector<int> ans(n);
    iota(all(ans), 0);
    vector<int> already = {1, 0, 2, 3};
    for (int i = 0; i < min(4, n); i++) {
        ans[i] = already[i];
    }
    for (int i = 1; i * 4 < n; i++) {
        swap(ans[i * 4 - 1], ans[i * 4]);
    }
    for (int i = 0; i < n; i++) {
        cout << ans[i] << ' ';
    }
    cout << '\n';
}
```
