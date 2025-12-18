---
title: Gifts from Knowledge
date: 2025-11-02
last_clear: 2025-11-02
tags:
  - QOJ/ICPC/Regionals/Jinan/2023
  - SUA
  - 红温
  - tag/模拟
  - tag/dsu
  - tag/细心
grasp: 3
value: 4
difficulty:
optional_difficulty: Easy-Medium
source: https://qoj.ac/contest/1472/problem/7900?v=1
article:
---
需要注意细节，不要想所谓的喵喵做法，老老实实的用二维vec存
```cpp showLineNumbers
void solve()
{
    int r, c;
    cin >> r >> c;
    vector<string> board(r);
    for (int i = 0; i < r; i++) {
        cin >> board[i];
    }
    DSU                 dsu(2 * r);
    vector<vector<int>> col1(c);
    vector<int>         cntarr(c, 0);
    for (int i = 0; i < r; i++) {
        for (int j = 0; j < c; j++) {
            if (board[i][j] == '1') cntarr[j]++;
        }
    }
    for (int i = 0; i < c; i++) {
        if (i == c / 2 && c % 2 == 1) continue;
        if (cntarr[i] + cntarr[c - i - 1] > 2) {
            cout << 0 << '\n';
            return;
        }
    }
    if (c % 2 == 1) {
        if (cntarr[c / 2] >= 2) {
            cout << 0 << '\n';
            return;
        }
    }
    for (int i = 0; i < r; i++) {
        for (int j = 0; j < c; j++) {
            if (j == c / 2 && c % 2 == 1) continue;
            if (board[i][j] == '1') {
                for (int v : col1[c - j - 1]) {
                    if (v == i) continue;
                    dsu.merge(i, v);
                    dsu.merge(i + r, v + r);
                }
                for (int v : col1[j]) {
                    if (v == i) continue;
                    dsu.merge(i, v + r);
                    dsu.merge(i + r, v);
                }
                col1[j].pb(i);
            }
        }
    }
    int cnt = dsu.component / 2;
    for (int i = 0; i < r; i++) {
        if (dsu.same(i, i + r)) {
            cout << 0 << '\n';
            return;
        }
    }

    cout << power(2, cnt) << '\n';
}
```