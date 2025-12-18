---
title: Stop, Yesterday Please No More
date: 2025-11-04
last_clear: 2025-11-04
tags:
  - QOJ/ICPC/Regionals/Nanjing/2022
  - SUA
  - tag/math/fftntt
  - tag/差分与前缀和
  - tag/二维前缀和
  - tag/thinking/约束分析
  - tag/thinking/reverse
grasp: 3
value: 5
difficulty:
optional_difficulty: Easy-Medium
source: https://qoj.ac/contest/1093
article:
---
将袋鼠的移动对称变化成井盖移动
同时注意到，最终剩余袋鼠一定是矩形(约束分析)，可以干二维前缀和
```cpp showLineNumbers
void solve()
{
    int n, m, k;
    cin >> n >> m >> k;
    string s;
    cin >> s;
    pair<int, int>      row{n, n}, col{m, m};
    int                 x = n, y = m;
    int                 xx = n, yy = m;
    vector<vector<int>> board(2 * n + 1, vector<int>(2 * m + 1));
    board[x][y] = 1;
    bool em     = 0;
    for (char c : s) {
        switch (c) {
            case 'U': {
                x--;
                xx++;
                break;
            }
            case 'D': {
                x++;
                xx--;
                break;
            }
            case 'L': {
                y--;
                yy++;
                break;
            }
            case 'R': {
                y++;
                yy--;
                break;
            }
        }
        row.fi = min(row.fi, x);
        row.se = max(row.se, x);
        col.fi = min(col.fi, y);
        col.se = max(col.se, y);
        if (row.se - row.fi >= n || col.se - col.fi >= m) {
            em = 1;
            break;
        }
        board[xx][yy] = 1;
    }
    if (em || (row.se - row.fi >= n) || (col.se - col.fi >= m)) {
        if (k == 0) {
            cout << n * m << '\n';
        } else {
            cout << 0 << '\n';
        }
        return;
    }
    for (int i = 1; i <= 2 * n; i++)
        for (int j = 1; j <= 2 * m; j++) board[i][j] += board[i][j - 1];

    for (int j = 1; j <= 2 * m; j++)
        for (int i = 1; i <= 2 * n; i++) board[i][j] += board[i - 1][j];
    int ans       = 0;
    i64 needtodie = (n - (row.se - row.fi)) * (m - (col.se - col.fi)) - k;
    dbg(board);
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            int L = n - (row.se - row.fi) - 1, R = m - (col.se - col.fi) - 1;
            int l = n - row.fi, r = m - col.fi;
            i64 die = board[i + L + l][j + R + r] - board[i + L + l][j - 1 + r] - board[i - 1 + l][j + R + r]
                      + board[i - 1 + l][j - 1 + r];
            if (die == needtodie) {
                ans++;
            }
        }
    }
    cout << ans << '\n';
}
```

我的猎奇NTT匹配：
```cpp showLineNumbers
using PolyZ = Poly<P>;

// floor(p / i) * i + p % i == p
// floor(p / i) * i + p % i == 0 (mod p)
// inv[i] == -inv[p % i] * floor(p / i)

void solve()
{
	// 很明显，在袋鼠没有全部跳完的情况下，洞的活动范围是一个在2n * 2m的矩形。
	// 那么就可以像前一个问题一样，尝试对每个点进行匹配，但是这样时间会来到n^4
	// 而对于一个固定大小的矩形进行匹配，可以将其展开成一维，因此就将袋鼠存在的位置和洞口位置进行累乘，最后统计合法范围内为target的数的数量即可
    int n, m, k;
    cin >> n >> m >> k;
    auto   zip   = [m](int x, int y) { return y * (2 * m) + x; };  // [0, 2 * n * m - 1]
    auto   unzip = [m](i64 c) -> pair<int, int> { return pair<int, int>(c % (2 * m), c / (2 * m)); };
    int    x = 0, y = 0;
    int    mx_x = 0, mx_y = 0, mn_x = 0, mn_y = 0;
    string s;
    cin >> s;
    for (char c : s) {
        switch (c) {
            case 'U': y--; break;
            case 'D': y++; break;
            case 'L': x--; break;
            case 'R': x++; break;
        }
        mx_x = max(mx_x, x);
        mn_x = min(mn_x, x);
        mx_y = max(mx_y, y);
        mn_y = min(mn_y, y);
    }
    if (mx_x - mn_x >= m || mx_y - mn_y >= n) {
        if (k == 0) {
            cout << n * m << '\n';
        } else {
            cout << 0 << '\n';
        }
        return;
    }
    PolyZ animo(4 * n * m + 1);
    PolyZ px(4 * n * m + 1);
    for (int iy = -mn_y; iy < n - mx_y; iy++) {
        for (int ix = -mn_x; ix < m - mx_x; ix++) {
            animo[zip(ix + m, iy + n)] = 1;
        }
    }
    int death = (n - mx_y + mn_y) * (m - mx_x + mn_x) - k;
    x = m, y = n;
    px[4 * n * m - zip(x, y)] = 1;
    for (char c : s) {
        switch (c) {
            case 'U': y++; break;
            case 'D': y--; break;
            case 'L': x++; break;
            case 'R': x--; break;
        }
        px[4 * n * m - zip(x, y)] = 1;
    }

    auto                res = std::move(animo) * std::move(px);
    int                 ans = 0;
    vector<vector<int>> tmp(n, vector<int>(m));
    for (int c = 4 * n * m; c < 6 * n * m; c++) {
        int j         = c - 4 * n * m;
        auto [ax, ay] = unzip(j);
        if (ax < 0 || ay < 0 || ax >= m || ay >= n) continue;
        tmp[ay][ax] = res[c].val();
        if (res[c] != death) continue;
        ans++;
    }
    dbg(tmp);
    cout << ans << '\n';
}
```
