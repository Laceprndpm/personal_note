---
title: " C. 造桥与砍树"
date: 2025-09-22
last_clear: 2025-09-22
tags:
  - tag/图论/生成树问题/最小生成树
grasp: 4
value: 4
difficulty:
optional_difficulty: 17.5%
source: https://qoj.ac/contest/2534/problem/14549
article:
---
# 题意
农夫的好帮手——乔冶，他喜欢在游戏里造桥和砍树。
乔冶在玩一款休闲模拟类游戏。他在游戏中生活的地区是一片群岛，具体的来说，一共有 $n$ 座岛屿，而乔冶一开始生活在 $1$ 号岛屿。岛屿和岛屿之间互不连通。
游戏中，第 $i$ 座岛屿有 $t_i$ 棵树，每砍伐一棵树就可以获得一根圆木。由于游戏的刷新机制，所有的岛屿在每天早上都会刷新新的树，也就是说，每天早上第 $i$ 座岛屿上的树的数量都会被设置为 $t_i$ 棵。
乔冶每天可以修建一座桥，以将任意两个岛屿连通（当然这两个岛屿必须至少有一个是他当前可以到达的）。他希望能尽快将所有岛屿连通，显然，这需要花费 $n-1$ 天。造桥本身不需要消耗圆木，乔冶有其他材料可用，可是他有一个爱好，是砍树。当他修建了连接岛屿 $u$ 和 $v$ 的桥时，他一定会在造桥的当天把岛屿 $u$ 和 $v$ 上的树全部砍光，共获得 $t_u + t_v$ 根圆木。（当然，这天过后岛屿上的树又会恢复）
游戏里有一个合成公式，每使用 $k$ 根圆木，就可以合成一根硬木，这个合成操作每天可以做任意次数。合成的硬木可以在每天晚上卖给市场获得金钱，但是剩余的圆木每天晚上都会被系统清空。乔冶不在乎自己能获取多少金钱，但是患有强迫症的他很讨厌自己辛苦获得的圆木被浪费掉。
所以，现在你需要帮助他解答这个问题：请你安排好每天的造桥计划，使得第 $n-1$ 天后所有岛屿可以相互到达，并且最小化这 $n-1$ 天里被系统清空的圆木总数，你只需要输出这个最小总数即可。
# Solution
首先，注意到他的路径代价和先前状态无关，所以肯定可以用 `prim` 算法跑一遍最小生成树
随后我们就获得了 $n^2$ 条边，我们就获得了一个 $O(n^2)$ 的算法
我们注意到我们还没有使用他的 边权=点权和模k 的性质，随后发现他可以画出一个这样的图
![[qq_pic_merged_1758380393454.jpg]]
上面的线段的每个点表示没有选择的点的权值
下面的线段的每个点表示已经选择了点的权值
上段的颜色表示该段上的每个点与下段的对应颜色的点组合时最优
但是，假如我们直接一次性标记每个未选择点应该和哪些点连接，要么代码相当难写，要么可能会 TLE
因此，我就把离每个已选择的点负责的区间最左面加入堆中，依次更新
同时，因为存在 $a_i + a_j < k$ 的情况，所以也要将其与最左面的点连边 (但实际上应该只要在已选择点的点权的最小值被更新时加点即可)
关于一些细节：
综上，我们队一开始只用了一个 `pair<int, int>` 来存储 "边权"，和"未选择点"。但后来发现，因为我们每次只将最近的未选择点与每个已选择点连边，所以样例二中的 `5 2 2` 这个 5 和一个 2 配完后就会消失，不符合我们的意图。所以改成了 `tuple<int, int, int>`来存储"边权"，"未选择点"，"已选择点"，即"val"，"to"，"from"。
```cpp showLineNumbers title="赛时AC"

#include <bits/stdc++.h>
using namespace std;
using i64 = long long;
constexpr i64 INF = 1e12;
void solve()
{
    int n, k;
    cin >> n >> k;
    vector<i64> ai(n + 1);
    for (int i = 1; i <= n; i++)
    {
        cin >> ai[i];
        ai[i] %= k;
    }
    if (n == 1)
    {
        cout << 0 << '\n';
        return;
    }
    multiset<i64> idx;
    for (int i = 2; i <= n; i++)
    {
        idx.insert(ai[i]);
    }
    priority_queue<tuple<i64,i64, i64>, vector<tuple<i64,i64, i64>>,greater<>> pq;

    pq.push({(*idx.begin() + ai[1]) % k, *idx.begin(), ai[1]});
    auto it = idx.lower_bound(k - ai[1]);
    if (it != idx.end())
    {
        pq.push({int((*it + ai[1]) % k), *it, ai[1]});
    }
    int visitednum = 1;
    i64 ans = 0;
    while (visitednum < n)
    {
        auto [val, id, who] = pq.top();
        pq.pop();

        if (!idx.contains(id)) continue;
        idx.extract(id);
        ans += val;
        visitednum++;
        if (visitednum >= n) break;
        pq.push({(*idx.begin() + id) % k, *idx.begin(), id});
        auto it = idx.lower_bound(k - id);;
        if (it != idx.end())
        {
            pq.push({((*it + id) % k), *it, id});
        }
        pq.push({(*idx.begin() + who) % k, *idx.begin(), who})
        ;
        auto it2 = idx.lower_bound(k - who);;
        if (it2 != idx.end())
        {
            pq.push({((*it2 + who) % k), *it2, who});
        }
    }
    cout << ans << '\n';
}
int main()
{
    int T;
    cin >> T;
    while (T--)
    {
        solve();
    }
}
```
