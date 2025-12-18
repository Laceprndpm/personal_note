---
title: M. Inverted
date: 2025-09-30
last_clear: 2025-09-30
tags:
  - tag/图论/树上问题/树上dp
  - tag/etc/卡常
grasp: 3
value: 5
difficulty:
optional_difficulty: Silver
source: https://qoj.ac/contest/1411/problem/7631
article:
---
# 卡常部分
- `lambda` 转函数提速1倍
- 尽可能的记忆化提速1倍
- 前缀/后缀积来去掉逆元计算提速2倍
- 预处理逆元提速不确定，但有些许提升
# 题意
# Solution
对每一个联通快，作为一棵树，选择一个根后，对所有子树dfs时，需要保证其联通，那么对每个儿子有两种情况：
1. 该处断开，需要儿子内部与原图连接
2. 该处保持连接
最后，需要保证整棵树与原图恰有一个连接
因此，设 `dp[u][c]` 为 `u` 的子树中，有 `c` 条路径连接到原图的方案数，每次转移需要保证当前子树最多不超过一条路径连到原图，且不可切断 `dp[u][0]` ，因为其需要连接到根，由其他路径连接到原图，最后每棵树中 `dp[u][1]` 即是最终答案。
需要注意的是，每条边可以选择割上面也可以选择割下面，两者无区别，所以乘以 $power(2,cnt)$
```cpp showLineNumbers
#include <array>
#include <cassert>
#include <iostream>
#include <numeric>
#include <vector>

using namespace std;
using ll   = long long;
using u8   = uint8_t;
using u16  = uint16_t;
using u32  = uint32_t;
using i64  = long long;
using u64  = uint64_t;
using i128 = __int128;
using u128 = unsigned __int128;
using f128 = __float128;

// vectors
#define sz(x)   int(size(x))
#define bg(x)   begin(x)
#define all(x)  bg(x), end(x)
#define rall(x) rbegin(x), rend(x)
#define sor(x)  sort(all(x))
#define rsz     resize
#define ins     insert
#define pb      push_back
#define eb      emplace_back
#define ft      front()
#define bk      back()
#define mt      make_tuple
#define fi      first
#define se      second

struct DSU {
    std::vector<int> f, siz;

    DSU() {}

    DSU(int n) { init(n); }

    void init(int n)
    {
        f.resize(n);
        std::iota(f.begin(), f.end(), 0);
        siz.assign(n, 1);
    }

    int find(int x)
    {
        while (x != f[x]) {
            x = f[x] = f[f[x]];
        }
        return x;
    }

    bool same(int x, int y) { return find(x) == find(y); }

    bool merge(int x, int y)
    {
        x = find(x);
        y = find(y);
        if (x == y) {
            return false;
        }
        siz[x] += siz[y];
        f[y] = x;
        return true;
    }

    int size(int x) { return siz[find(x)]; }
};

constexpr int P = 998244353;

constexpr u64 power(u64 a, u64 b)
{
    u64 res = 1;
    for (; b != 0; b /= 2, a = a * a % P) {
        if (b & 1) {
            res = (res * a) % P;
        }
    }
    return res;
}

constexpr int optnum   = 2e5;
vector<int>   cacheopt = [](int _init = optnum) -> vector<int> {
    vector<int> vec(optnum);
    vec[1] = 1;
    for (int i = 2; i < optnum; i++) {
        vec[i] = ((u64)(P - P / i) * vec[P % i]) % P;
    }
    return vec;
}();

constexpr u64 inv(u64 x)
{
    if (x < optnum) {
        return cacheopt[x];
    } else {
        return power(x, P - 2);
    }
}

vector<array<u32, 2>> dp;
vector<vector<int>>   graph;
vector<vector<int>>   newgraph;
vector<int>           vis;
int                   cnt = 0;

u32 prefixbuf[int(5e3) + 50];
u32 suffixbuf[int(5e3) + 50];

void dfs(int u, int fa)
{
    if (!vis[u]) {
        cnt++;
        dp[u][1] = 1;
        dp[u][0] = 0;
        return;
    }
    u64 cdp1  = 0;
    int sonsz = newgraph[u].size();
    for (int i = 0; i < sonsz; i++) {
        int v = newgraph[u][i];
        if (v == fa) continue;
        dfs(v, u);
    }
    prefixbuf[0]         = 1;
    suffixbuf[sonsz - 1] = 1;
    for (int i = 0; i < sonsz; i++) {
        int v = newgraph[u][i];
        if (v == fa) {
            prefixbuf[i + 1] = prefixbuf[i];
            continue;
        }
        prefixbuf[i + 1] = u64(prefixbuf[i]) * (dp[v][0] + dp[v][1]) % P;
    }
    for (int i = sonsz - 1; i >= 1; i--) {
        int v = newgraph[u][i];
        if (v == fa) {
            suffixbuf[i - 1] = suffixbuf[i];
            continue;
        }
        suffixbuf[i - 1] = u64(suffixbuf[i]) * (dp[v][0] + dp[v][1]) % P;
    }
    for (int i = 0; i < sonsz; i++) {
        int v = newgraph[u][i];
        if (v == fa) continue;
        cdp1 = (cdp1 + u64(prefixbuf[i]) * suffixbuf[i] % P * dp[v][1]) % P;
    }
    dp[u][0] = prefixbuf[sonsz], dp[u][1] = cdp1;
};

void solve()
{
    int n;
    cin >> n;
    if (n == 1) {
        cout << '\n';
        return;
    }
    dp.resize(n + 1);
    graph.resize(n + 1);
    newgraph.resize(n + 1);
    vis.resize(n + 1);
    DSU         dsu(n + 1);
    vector<int> dsucnt(n + 1);
    for (int i = 1; i <= n - 1; i++) {
        int u, v;
        cin >> u >> v;
        graph[u].eb(v);
        graph[v].eb(u);
    }
    u64 ans = 1;
    for (int i = 1; i <= n - 1; i++) {
        int u;
        cin >> u;
        vis[u] = 1;
        for (int adj : graph[u]) {
            if (!vis[adj]) {
                newgraph[u].pb(adj);
                newgraph[adj].pb(u);
            } else {
                ans = ans * inv(dp[dsu.find(adj)][1]) % P;
                ans = ans * inv(power(2, dsucnt[dsu.find(adj)])) % P;
                dsu.merge(u, adj);
            }
        }
        cnt = 0;
        cnt--;
        dfs(u, -1);
        ans       = ans * dp[u][1] % P;
        dsucnt[u] = cnt;
        ans       = ans * power(2, cnt) % P;
        cout << ans << '\n';
    }
}

signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
#ifdef BATCH
    freopen(argv[1], "r", stdin);
    freopen(argv[2], "w", stdout);
#endif
    int t = 1;
    while (t--) {
        solve();
    }
    return 0;
}

/* stuff you should look for
 * int overflow, array bounds
 * special cases (n=1?)
 * do smth instead of nothing and stay organized
 * WRITE STUFF DOWN
 * DON'T GET STUCK ON ONE APPROACH
 */
```
