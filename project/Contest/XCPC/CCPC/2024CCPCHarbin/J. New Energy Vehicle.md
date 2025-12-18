---
title: New Energy Vehicle
date: 2025-10-21
last_clear:
tags:
  - QOJ/UCUP/3rd/Stage_14_Harbin
  - tag/贪心
grasp: 4
value: 4
difficulty:
optional_difficulty: Medium
source: https://codeforces.com/group/iF1v4Ff8lz/contest/631373/problem/J
article:
---
很明显的一个贪心：
我希望下次加油可以尽可能的多装，我肯定按照油站的顺序使用。
但是如果有两个相同的油站同时出现，下一个油站需要在当前油站加完后再使用，如果直接pop可能会"错过"。
因此用pq维护下标，同时维护 `nxt` --下一个同名油站的下标。
```cpp showLineNumbers
void solve()
{
    i64 n, m;
    cin >> n >> m;
    vector<i64> arr(n + 1);
    i64         gas = 0;
    for (int i = 1; i <= n; i++) {
        cin >> arr[i];
    }
    gas = accumulate(all(arr), 0ll);
    vector<i64> xrr(m), trr(m);
    for (int i = 0; i < m; i++) {
        cin >> xrr[i] >> trr[i];
    }
    vector<int> tmp_last(n + 1, m);
    vector<int> nxtarr(m, m);
    for (int i = m - 1; i >= 0; i--) {
        nxtarr[i]        = tmp_last[trr[i]];
        tmp_last[trr[i]] = i;
    }
    priority_queue<int, vector<int>, greater<>> que;
    vector<i64>                                 curarr = arr;
    for (int i = 0; i < m; i++) que.push(i);
    i64 curx = 0;
    i64 ans  = -1;
    for (int i = 0; i < m; i++) {
        i64 dist = xrr[i] - curx;
        if (dist > gas) {
            ans = gas + curx;
            break;
        }
        while (dist) {
            if (!que.empty()) {
                int u = que.top();
                que.pop();
                if (u < i) continue;
                if (u == m) continue;
                i64 uval = curarr[trr[u]];
                if (uval > dist) {
                    que.push(u);
                    uval = dist;
                }
                curarr[trr[u]] -= uval;
                gas -= uval;
                dist -= uval;
            } else {
                gas -= dist;
                dist = 0;
            }
        }
        gas += arr[trr[i]] - curarr[trr[i]];
        curarr[trr[i]] = arr[trr[i]];
        que.push(nxtarr[i]);
        curx = xrr[i];
    }
    if (ans == -1) {
        ans = gas + curx;
    }
    cout << ans << '\n';
}
```
