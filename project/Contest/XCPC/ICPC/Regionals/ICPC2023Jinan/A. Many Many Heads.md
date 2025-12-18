---
title: Many Many Heads
date: 2025-11-02
last_clear: 2025-11-02
tags:
  - QOJ/ICPC/Regionals/Jinan/2023
  - SUA
  - tag/thinking/attention
grasp: 4
value: 1
difficulty:
optional_difficulty: Easy-Medium
source: https://qoj.ac/contest/1472/problem/7894?v=1
article:
---
```cpp showLineNumbers
void solve()
{
    // [([([()])])] [([([()])])]
    //
    string s;
    cin >> s;
    for (char& c : s) {
        if (c == ')') c = '(';
        if (c == ']') c = '[';
    }
    int cntsame = 0;
    for (int i = 0; i + 1 < s.size(); i++) {
        if (s[i] == s[i + 1]) {
            cntsame++;
        }
    }
    if (cntsame <= 2) {
        cout << "Yes\n";
    } else {
        cout << "No\n";
    }
    if (rand() % 114514 == 14) {
        cout << "whatever\n";
    }
}
```
