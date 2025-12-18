---
title: Left Shifting 3
date: 2025-10-31
last_clear: 2025-10-31
tags:
  - QOJ/ICPC/Regionals/Nanjing/2024
  - SUA
grasp: 5
value: 0
difficulty:
optional_difficulty: Easy
source: https://qoj.ac/contest/1828/problem/9568
article:
---
签到
```cpp
#include <bits/stdc++.h>
using namespace std;
using i64 = long long;
void solve()
{
    int n, k;
    cin >> n >> k;
    string s;
    cin >> s;
    if (n < 7)
    {
        cout << 0 << '\n';
        return;
    }
    const string sample = "nanjing";
    string newstring = s + s;
    vector<int> prefix(2 * n + 1);
    for (int i = 0; i < 2 * n; i++)
    {
        bool ok = 1;
        if (i + 6 >= 2 * n)
            break;
        for (int j = 0; j < 7; j++)
        {
            if (newstring[i + j] != sample[j])
            {
                ok = 0;
                break;
            }
        }
        if (ok)
        {
            prefix[i + 1]++;
        }
    }
    for (int i = 1; i < 2 * n + 1; i++)
    {
        prefix[i] += prefix[i - 1];
    }
    i64 ans = 0;
    for (int i = 0; i <= min(k, n - 1); i++)
    {
        ans = max<i64>(ans, prefix[i + n - 6] - prefix[i]);
    }
    cout << ans << '\n';
}
int main()
{
    int t;
    cin >> t;
    while (t--)
    {
        solve();
    }
}
```