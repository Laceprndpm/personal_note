---
title: Fuzzy Ranking
date: 2025-10-28
last_clear: 2025-10-28
tags:
  - QOJ/ICPC/Regionals/Hangzhou/2024
  - SUA
  - tag/图论/连通性相关/SCC
  - tag/thinking/attention
  - tag/差分与前缀和
grasp: 3
value: 4
difficulty:
optional_difficulty: Easy-Medium
source: https://qoj.ac/contest/1893/problem/9731
article: https://qoj.ac/download.php?type=attachments&id=1893&r=1
---
注意到每个颜色的块连续即可
因为如果存在 $A' \leftarrow B \leftarrow A$
那么明显由强联通分量性质 $A' \rightarrow A$，可以证明 $B\in A$

```cpp showLineNumbers
void solve()
{
    int n, k, q;
    cin >> n >> k >> q;
    SCC                 scc(n);
    vector<vector<int>> board(k, vector<int>(n));
    for (int i = 0; i < k; i++) {
        for (int j = 0; j < n; j++) {
            cin >> board[i][j];
            board[i][j]--;
            if (j) scc.addEdge(board[i][j], board[i][j - 1]);
        }
    }
    scc.work();
    vector<i64> cnt(n);
    for (int i = 0; i < k; i++) {
        for (int j = 0; j < n; j++) {
            board[i][j] = scc.bel[board[i][j]];
        }
    }
    for (int j = 0; j < n; j++) {
        cnt[board[0][j]]++;
    }
    vector<array<vector<int>, 2>> ext(k);

    vector<vector<i64>> prefix(k, vector<i64>(n + 1));
    for (int i = 0; i < k; i++) {
        vector<int> lft(n, n);
        vector<int> rgt(n, -1);
        for (int j = 0; j < n; j++) {
            lft[board[i][j]] = min(lft[board[i][j]], j);
            rgt[board[i][j]] = max(rgt[board[i][j]], j + 1);
        }
        int ptr = 0;
        while (ptr < n) {
            i64 tmpval = cnt[board[i][ptr]] * (cnt[board[i][ptr]] - 1ll) / 2;
            ptr        = rgt[board[i][ptr]];
            prefix[i][ptr] += tmpval;
        }
        ext[i] = {lft, rgt};
        for (int j = 0; j < n; j++) {
            prefix[i][j + 1] += prefix[i][j];
        }
    }
    i64 v = 0;
    while (q--) {
        i64 id, l, r;
        cin >> id >> l >> r;
        id                = (id + v) % k;
        l                 = (l + v) % n;
        r                 = (r + v) % n + 1;
        const auto &curbd = board[id];
        i64         ll = ext[id][1][curbd[l]], rr = ext[id][0][curbd[r - 1]];
        if (ll > rr) {
            v = (r - l) * (r - l - 1) / 2;
        } else {
            i64 midval = prefix[id][rr] - prefix[id][ll];
            i64 lftval = (r - rr) * (r - rr - 1) / 2;
            i64 rgtval = (ll - l) * (ll - l - 1) / 2;
            v          = lftval + midval + rgtval;
        }
        cout << v << '\n';
    }
}
```
