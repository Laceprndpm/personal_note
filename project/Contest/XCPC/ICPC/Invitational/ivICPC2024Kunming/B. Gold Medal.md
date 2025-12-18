---
title: Gold Medal
date: 2025-10-31
last_clear: 2025-10-31
tags:
  - QOJ/ICPC/Invitational/Kunming/2024
grasp: 5
value: 0
difficulty:
optional_difficulty: CheckIn
source: https://qoj.ac/contest/1802/problem/9423
article:
---

```cpp showLineNumbers
void solve()
{
    int n, k;
    cin >> n >> k;
    vector<int> arr(n);
    i64         curans = 0;
    for (int& i : arr) {
        cin >> i;
        curans += i / k;
        i %= k;
        i = k - i;
    }
    sor(arr);
    i64 m;
    cin >> m;
    for (int i : arr) {
        if (m >= i) {
            m -= i;
            curans++;
        }
    }
    curans += m / k;
    cout << curans << '\n';
}
```