---
title: Social Media
date: 2025-10-31
last_clear: 2025-10-31
tags:
  - QOJ/ICPC/Regionals/Nanjing/2024
  - SUA
  - tag/模拟
grasp: 4
value: 5
difficulty:
optional_difficulty: Easy
source: https://qoj.ac/contest/1828/problem/9573
article:
---
```cpp showLineNumbers
void solve()
{
    int n, m, k;
    cin >> n >> m >> k;
    vector<int> farr(n + 1);
    vector<int> isfrind(k + 1);
    for (int i = 1; i <= n; i++) {
        cin >> farr[i];
        isfrind[farr[i]] = 1;
    }
    vector<pair<int, int>> arr(m + 1);
    vector<vector<int>>    go(k + 1);
    int                    baseans = 0;
    vector<int>            bounds(k + 1);
    for (int i = 1; i <= m; i++) {
        cin >> arr[i].fi;
        cin >> arr[i].se;
        if (arr[i].fi > arr[i].se) swap(arr[i].fi, arr[i].se);
        if (isfrind[arr[i].fi] && isfrind[arr[i].se]) {
            baseans++;
        } else if (isfrind[arr[i].fi]) {
            bounds[arr[i].se]++;
        } else if (isfrind[arr[i].se]) {
            bounds[arr[i].fi]++;
        } else if (arr[i].fi == arr[i].se) {
            bounds[arr[i].fi]++;
        } else {
            go[arr[i].fi].pb(arr[i].se);
        }
    }
    LazySegmentTree<Info, Tag> lzt(bounds);
    i64                        ans = 0;
    for (int i = 1; i <= k; i++) {
        i64 cur = bounds[i];
        for (int v : go[i]) {
            lzt.update(v, 1);
        }
        ans = max(ans, baseans + cur + lzt.rangeQuery(i + 1, k + 1).x);
        for (int v : go[i]) {
            lzt.update(v, -1);
        }
    }
    cout << ans << '\n';
}
```
能过，但是很不优雅，需要线段树

可以用桶维护，因为不需要具体的位置
```cpp showLineNumbers
void solve()
{
    int n, m, k;
    cin >> n >> m >> k;
    vector<int> isfrind(k + 1);
    for (int i = 1, has; i <= n; i++) {
        cin >> has;
        isfrind[has] = 1;
    }
    vector<vector<int>> go(k + 1);
    int                 baseans = 0;
    vector<int>         bounds(k + 1);
    for (int i = 1, u, v; i <= m; i++) {
        cin >> u >> v;
        if (u > v) swap(u, v);
        if (isfrind[u] && isfrind[v]) {
            baseans++;
        } else if (isfrind[u]) {
            bounds[v]++;
        } else if (isfrind[v]) {
            bounds[u]++;
        } else if (u == v) {
            bounds[u]++;
        } else {
            go[u].pb(v);
        }
    }
    if (n + 2 >= k) {
        cout << m << '\n';
        return;
    }
    i64         ans = 0;
    vector<int> peoplecnt(m + 2);  // 每个贡献对应人数
    for (int i = 1; i <= k; i++) peoplecnt[bounds[i]]++;
    int curmax = m + 1;
    while (peoplecnt[curmax] == 0) curmax--;
    auto update = [&](int u, int val) {
        peoplecnt[bounds[u]]--;
        bounds[u] += val;
        peoplecnt[bounds[u]]++;
        if (peoplecnt[curmax + 1] >= 1) curmax++;
        if (peoplecnt[curmax] == 0) curmax--;
    };
    for (int i = 1; i <= k; i++) {
        i64 cur = bounds[i];
        peoplecnt[cur]--;
        int bak = curmax;
        while (peoplecnt[curmax] == 0) curmax--;
        for (int v : go[i]) update(v, 1);
        ans = max(ans, baseans + cur + curmax);
        for (int v : go[i]) update(v, -1);
        peoplecnt[cur]++;
        curmax = bak;
    }
    cout << ans << '\n';
}
```