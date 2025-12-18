---
title: Perfect Journey
date: 2025-07-29
last_clear: 2025-07-29
tags:
  - tag/计数/容斥
  - tag/math/fwt
  - tag/dp/sosdp
grasp: 2
difficulty: 2000
source: https://ac.nowcoder.com/acm/contest/108302/K
article: https://www.cnblogs.com/cyl06/p/SOSDP.html
article1: https://www.cnblogs.com/Pbriqwq/p/15429971.html
article2: file:///home/patchouli/Voile/project/2025NkSummer/attachment/2025NkSummer/5/2025_%E7%89%9B%E5%AE%A2%E6%9A%91%E6%9C%9F%E5%A4%9A%E6%A0%A1%E8%AE%AD%E7%BB%83%E8%90%A55%E9%A2%98%E8%A7%A3.pdf
---
# 题意
给定一棵 $n$ 个结点的树，以及 $m$ 条关键边，$k$ 条树上路径。  
需要从中选出尽量少的路径，使得所有关键边都被经过。  

输出：  
- 最少选择路径的数量；  
- 达到最少数量的方案数。  

约束条件：  
- $3 \le n \le 2 \times 10^5$  
- $1 \le m \le 22$  
- $1 \le k \le 2 \times 10^5$
# Solution
## san_cai:
```cpp showLineNumbers
#include <iostream>
#include <algorithm>
#include <cstring>
#include <vector>
#define endl '\n'
#define ios ios::sync_with_stdio(false), cin.tie(nullptr), cout.tie(nullptr)
using namespace std;
typedef long long LL;

const int N = 2e5 + 10;
const int M = 22;
const int MOD = 998244353;

int n, m, k;
vector<pair<int, int>> g[N];
int sm[N];
int fa[N][20], dep[N];
int msk[N];

LL cnt[1 << M];
LL sos[1 << M];
LL fact[N], infact[N];

LL qmi(LL a, LL b) {
    LL res = 1;
    while (b) {
        if (b & 1) res = (LL) res * a % MOD;
        a = (LL) a * a % MOD;
        b >>= 1;
    }
    return res;
}

LL inv(LL a) {
    return qmi(a, MOD - 2);
}

void init(int max_val) {
    fact[0] = 1;
    infact[0] = 1;
    for (int i = 1; i <= max_val; i++) {
        fact[i] = (fact[i - 1] * i) % MOD;
        infact[i] = inv(fact[i]);
    }
}

LL C(int n, int k) {
    if (k < 0 || k > n) return 0;
    return fact[n] * infact[k] % MOD * infact[n - k] % MOD;
}

void dfs_lca(int u, int p, int d, int msks) {
    dep[u] = d;
    fa[u][0] = p;
    msk[u] = msks;
    for (auto& e : g[u]) {
        int v = e.first;
        int idx = e.second;
        if (v == p) continue;
        int nxt = msks;
        if (sm[idx] != -1) {
            nxt ^= (1 << sm[idx]);
        }
        dfs_lca(v, u, d + 1, nxt);
    }
}

void build() {
    memset(fa, 0, sizeof(fa));
    memset(dep, 0, sizeof(dep));
    memset(msk, 0, sizeof(msk));
    dfs_lca(1, 0, 1, 0);
    for (int j = 1; j < 20; ++j) {
        for (int i = 1; i <= n; ++i) {
            fa[i][j] = fa[fa[i][j - 1]][j - 1];
        }
    }
}

void solve() {
    cin >> n >> m >> k;

    for (int i = 0; i <= n; ++i) g[i].clear();
    memset(sm, -1, sizeof(sm));
    memset(cnt, 0, sizeof(cnt));

    for (int i = 1; i < n; ++i) {
        int u, v;
        cin >> u >> v;
        g[u].emplace_back(v, i);
        g[v].emplace_back(u, i);
    }

    for (int i = 0; i < m; ++i) {
        int x;
        cin >> x;
        sm[x] = i;
    }

    build();

    int ms = 0;
    for (int i = 0; i < k; ++i) {
        int u, v;
        cin >> u >> v;
        int tmp = msk[u] ^ msk[v];
        cnt[tmp]++;
        ms |= tmp;
    }

    if (ms != (1 << m) - 1) {
        cout << -1 << endl;
        return;
    }

    memcpy(sos, cnt, sizeof(sos));
    for (int i = 0; i < m; ++i) {
        for (int mask = 0; mask < (1 << m); ++mask) {
            if (mask & (1 << i)) {
                sos[mask] += sos[mask ^ (1 << i)];
            }
        }
    }

    vector<LL> dp(1 << m);
    for (int mask = 0; mask < (1 << m); ++mask) {
        int bits = __builtin_popcount(mask);
        if ((m - bits) % 2 == 0) {
            dp[mask] = 1;
        } else {
            dp[mask] = MOD - 1;
        }
    }

    for (int c = 1; c <= m; ++c) {
        LL res = 0;
        for (int mask = 0; mask < (1 << m); ++mask) {
            LL tim = C(sos[mask], c);
            res = (res + dp[mask] * tim) % MOD;
        }
        if (res != 0) {
            cout << c << " " << res << endl;
            return;
        }
    }

    cout << -1 << endl;
}
```

![[Pasted image 20250802231346.png]]
[2025年 08月 03日 星期日 17:09:37 CST]
解释：
设 $a_1, a_2, \dots a_n$ 为经过该边的性质
我们要计算
$$
N(a_1a_2\dots a_n) = \sum_{i = 0}^n(-1)^i \sum_{1\le j_1 < j_2 < \dots < j_i \le n}N((1-a_{j_1}) \dots (1-a_{j_i}))
$$
假设选择了 $c$ 条路径
$$
N((1-a_{j_1}) \dots (1-a_{j_i})) =N\Big(\prod_{j \in J} (1 - a_j)\Big) = \binom{cnt路径_{不包含j \in J}}{c}
$$
设 $s >> i \; \& \; 1$ 表示**可能**含有 $i$ 号边(即 $!(s >> i \; \& 1)$ 是确定不包含 $i$ )，则:
$$
\sum (-1)^{n-\text{popcount}(s)} \cdot \binom{dp[s]}{c}
$$
**Q**:为什么需要反转？
**A**:可能是因为 $sosdp$ 只能够统计超集，或者说想计算 $cnt_{不包含}$ ，必须要进行该操作。或者更直白的说，套模板。
感觉和 [P3160 [CQOI2012] 局部极小值](https://www.luogu.com.cn/problem/P3160) 高度相似，均为超集上进行容斥
详见[[P3160]]
## std
#TODO/快速莫比乌斯变换
可以用状压来表示经过边的情况，这样问题等价于：  
选择尽量少的路径，使得这些路径覆盖的边集合并起来恰为全集。  

可以考虑使用 FWT-OR 来解决这一问题。  
- 直接计算的复杂度是 $O(m \cdot 2^{2m})$。  
- 但我们实际上只关心全集对应项的系数，因此只需通过容斥来计算该项。  

记  
- $mask = 2^m - 1$  
- $f$ 为做 $t$ 次 FWT 之后得到的多项式  

则全集项系数为  
$$
\sum_{i=0}^{2^m - 1} (-1)^{\mathrm{pop\_cnt}(i \oplus mask)\ \&\ 1} \cdot f_i
$$  

整体复杂度为 $O(m \cdot 2^m)$。  
此外，还有依赖于树结构的 $O(m \cdot 2^{2m})$ 与 $O(m \cdot 2^m)$ 做法，前者可能无法通过本题。  
mycode:
TLE
```cpp showLineNumbers
void zeta(vector<Z> &f)
{
    int n = __builtin_ctz(f.size());
    for (int i = 0; i < n; ++i)
        for (int mask = 0; mask < (1 << n); ++mask)
            if (!(mask & (1 << i))) f[mask | (1 << i)] += f[mask];
}

void mobius(vector<Z> &f)
{
    int n = __builtin_ctz(f.size());
    for (int i = 0; i < n; ++i)
        for (int mask = 0; mask < (1 << n); ++mask)
            if (!(mask & (1 << i))) f[mask | (1 << i)] -= f[mask];
}

void Or(vector<Z> &f, ll type)  // 迭代实现，常数更小
{
    int n = sz(f);
    for (ll x = 2; x <= n; x <<= 1) {
        ll k = x >> 1;
        for (ll i = 0; i < n; i += x) {
            for (ll j = 0; j < k; j++) {
                f[i + j + k] += f[i + j] * type;
            }
        }
    }
}

vector<Z> calc(const vector<Z> &a, const vector<Z> &b)
{
    int       n  = a.size();
    vector<Z> fa = a, fb = b;
    Or(fa, 1);
    Or(fb, 1);
    for (int i = 0; i < n; ++i) fa[i] *= fb[i];
    Or(fa, Z::mod() - 1);
    return fa;
}

void solve()
{
    int n, m, k;
    cin >> n >> m >> k;
    vector<pair<int, int>> edge(n);

    vector<vector<int>> graph(n + 1);
    for (int i = 1; i <= n - 1; i++) {
        int u, v;
        cin >> u >> v;
        graph[u].eb(v);
        graph[v].eb(u);
        edge[i] = {u, v};
    }
    vector<int> tree_route(n + 1);
    vector<int> depth(n + 1);

    auto dfs = [&](auto &&dfs, int u, int fa) -> void {
        for (int v : graph[u]) {
            if (v == fa) continue;
            depth[v] = depth[u] + 1;
            dfs(dfs, v, u);
        }
    };

    depth[1] = 0;
    dfs(dfs, 1, 1);

    for (int i = 0; i < m; i++) {
        int which;
        cin >> which;
        auto [u, v] = edge[which];
        if (depth[u] < depth[v]) swap(u, v);
        tree_route[u] |= 1 << i;
    }
    auto dfs2 = [&](auto &&dfs2, int u, int fa) -> void {
        for (int v : graph[u]) {
            if (v == fa) continue;
            tree_route[v] |= tree_route[u];
            dfs2(dfs2, v, u);
        }
    };
    dfs2(dfs2, 1, 1);

    vector<pair<int, int>> vec(k);
    for (int i = 0; i < k; i++) {
        int s, t;
        cin >> s >> t;
        vec[i] = {s, t};
    }

    vector<Z> dp1(1 << (m));

    for (int i = 0; i < k; i++) {
        int res = tree_route[vec[i].fi] ^ tree_route[vec[i].se];
        dbg(res);
        dp1[res] += 1;
    }
    const int mask = (1 << m) - 1;
    auto      cur  = dp1;
    for (int i = 1; i < 22; i++) {
        Z ans = cur[(1 << m) - 1];
        if (ans != 0) {
            cout << i << ' ' << ans / comb.fac(i) << '\n';
            return;
        }
        dbg(cur);
        cur = calc(cur, dp1);
        dbg(cur);
    }
    cout << -1 << '\n';
}
```
mycode:
AC
```cpp showLineNumbers
void zeta(vector<Z> &f)
{
    int n = __builtin_ctz(f.size());
    for (int i = 0; i < n; ++i)
        for (int mask = 0; mask < (1 << n); ++mask)
            if (!(mask & (1 << i))) f[mask | (1 << i)] += f[mask];
}

void mobius(vector<Z> &f)
{
    int n = __builtin_ctz(f.size());
    for (int i = 0; i < n; ++i)
        for (int mask = 0; mask < (1 << n); ++mask)
            if (!(mask & (1 << i))) f[mask | (1 << i)] -= f[mask];
}

void Or(vector<Z> &f, ll type)  // 迭代实现，常数更小
{
    int n = sz(f);
    for (ll x = 2; x <= n; x <<= 1) {
        ll k = x >> 1;
        for (ll i = 0; i < n; i += x) {
            for (ll j = 0; j < k; j++) {
                f[i + j + k] += f[i + j] * type;
            }
        }
    }
}

vector<Z> calc(const vector<Z> &a, const vector<Z> &b)
{
    int       n  = a.size();
    vector<Z> fa = a, fb = b;
    Or(fa, 1);
    Or(fb, 1);
    for (int i = 0; i < n; ++i) fa[i] *= fb[i];
    Or(fa, Z::mod() - 1);
    return fa;
}

void solve()
{
    int n, m, k;
    cin >> n >> m >> k;
    vector<pair<int, int>> edge(n);

    vector<vector<int>> graph(n + 1);
    for (int i = 1; i <= n - 1; i++) {
        int u, v;
        cin >> u >> v;
        graph[u].eb(v);
        graph[v].eb(u);
        edge[i] = {u, v};
    }
    vector<int> tree_route(n + 1);
    vector<int> depth(n + 1);

    auto dfs = [&](auto &&dfs, int u, int fa) -> void {
        for (int v : graph[u]) {
            if (v == fa) continue;
            depth[v] = depth[u] + 1;
            dfs(dfs, v, u);
        }
    };

    depth[1] = 0;
    dfs(dfs, 1, 1);

    for (int i = 0; i < m; i++) {
        int which;
        cin >> which;
        auto [u, v] = edge[which];
        if (depth[u] < depth[v]) swap(u, v);
        tree_route[u] |= 1 << i;
    }
    auto dfs2 = [&](auto &&dfs2, int u, int fa) -> void {
        for (int v : graph[u]) {
            if (v == fa) continue;
            tree_route[v] |= tree_route[u];
            dfs2(dfs2, v, u);
        }
    };
    dfs2(dfs2, 1, 1);

    vector<pair<int, int>> vec(k);
    for (int i = 0; i < k; i++) {
        int s, t;
        cin >> s >> t;
        vec[i] = {s, t};
    }

    vector<Z> dp1(1 << (m));

    for (int i = 0; i < k; i++) {
        int res = tree_route[vec[i].fi] ^ tree_route[vec[i].se];
        dbg(res);
        dp1[res] += 1;
    }
    const int mask = (1 << m) - 1;
    auto      cur  = dp1;
    Or(cur, 1), Or(dp1, 1);
    for (int i = 1; i <= m; i++) {
        Z ans = 0;
        for (int j = 0; j < 1 << m; j++) {
            ans += (__builtin_popcount(mask ^ j) & 1 ? -1 : 1) * cur[j];
        }
        if (ans != 0) {
            cout << i << ' ' << ans / comb.fac(i) << '\n';
            return;
        }
        dbg(cur);
        // cur = calc(cur, dp1);
        for (int j = 0; j < 1 << m; j++) {
            cur[j] *= dp1[j];
        }
        dbg(cur);
    }
    cout << -1 << '\n';
}
```
## xll
S 是求和的指标，就是 {0, 1, ..., m - 1} 的子集
就比如 S 是空集的话，表示的是所有路线的个数，因为此时没有任何约束
然后 是不是其实 m 条路线一定是可以覆盖 m 条关键道路的，否则一定是无解的
那好像 可以 m * (2^m)
先想办法把 h[S] 算出来
就全都预处理一遍
然后我们从 r = 0 到 r = m，算上面那个表达式，找到第一个 > 0 的，就能得到最少旅游路线数和方案数
m * (2 ** m) 应该 能过的
我研究下 h[S] 要怎么计算
如果记 f[S] 是，走过的关键道路的集合恰好为 S 的路线的数量
那么 h[S] 其实就是 全集 {0, 1, ..., m - 1} 去掉 S 中元素以后，这个东西的所有的子集 T，对 f[T] 求和的结果
f[S] 是可以直接算出来的，这个不难
现在只需要 对一个集合，怎么求他的所有子集的 f 值的和
