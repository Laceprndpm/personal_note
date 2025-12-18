---
title: Drain the Water Tank
date: 2025-09-11
last_clear: 2025-11-04
tags:
  - QOJ/ICPC/Regionals/Nanjing/2022
  - SUA
  - tag/计算几何
grasp: 3
value: 4
difficulty:
optional_difficulty: Easy-Medium
source: https://qoj.ac/contest/1093/problem/5426
article:
---
需要亿点点细节的计算几何题，核心在于处理 -V- 和 $\text{\\\_ \_/}$ 
```cpp
void solve()
{
    int n;
    cin >> n;
    vector<pair<int, int>> graph(n + 2);
    for (int i = 1; i <= n; i++) {
        cin >> graph[i].fi >> graph[i].se;
    }
    int orign = 0;
    for (int i = 1; i < n; i++) {
        if (graph[i].se != graph[i + 1].se) {
            orign = i + 1;
            break;
        }
    }
    int num_p = n;
    for (int i = 0; i < n; i++) {
        int cur  = (orign + i) % n;
        int nxt  = (cur + 1) % n;
        int nnxt = (nxt + 1) % n;
        if (cur == 0) {
            cur = n;
        }
        if (nxt == 0) {
            nxt = n;
        }
        if (nnxt == 0) {
            nnxt = n;
        }
        if (graph[nxt].se == graph[cur].se && graph[nnxt].se == graph[cur].se) {
            graph[nxt] = graph[cur];
            num_p--;
        }
    }
    vector<pair<int, int>> newg(n + 2);
    for (int i = 0; i < n; i++) {
        int cur = (orign + i) % n;
        if (cur == 0) cur = n;
        newg[i + 1] = graph[cur];
    }
    graph = newg;
    graph.erase(unique(graph.begin() + 1, graph.end() - 1), graph.end() - 1);

    graph[0]         = graph[num_p];
    graph[num_p + 1] = graph[1];
    graph.pb(graph[2]);
    auto cha   = [](pair<int, int> a, pair<int, int> b) { return a.fi * b.se - a.se * b.fi; };
    auto check = [&](int id) {
        assert(graph[id - 1].se != graph[id].se);
        assert(graph[id + 1].se != graph[id].se);
        if (graph[id - 1].se - graph[id].se < 0 || graph[id + 1].se - graph[id].se < 0) return false;
        pair<int, int> fm = {graph[id].fi - graph[id - 1].fi, graph[id].se - graph[id - 1].se};
        pair<int, int> to = {graph[id + 1].fi - graph[id].fi, graph[id + 1].se - graph[id].se};
        assert(cha(fm, to));
        if (atan2(cha(fm, to), fm.fi * to.fi + fm.se * to.se) > 0) {
            return true;
        }
        return false;
    };
    int num = 0;
    for (int i = 1; i <= num_p; i++) {
        if (graph[i + 1].se == graph[i].se) {         // 如果下一个点水平了
            if (graph[i + 1].fi - graph[i].fi > 0) {  // 向右
                assert(graph[i + 2].se != graph[i + 1].se);
                if (graph[i - 1].se > graph[i].se && graph[i + 2].se > graph[i + 1].se) {  // 下下个上升
                    num++;
                }
            }
            i++;
            continue;
        }
        if (check(i)) {
            num++;
        }
    }
    cout << num << '\n';
}

```