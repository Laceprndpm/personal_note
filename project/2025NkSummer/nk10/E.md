---
title: Sensei and Affection (affection)
date: 2025-09-06
last_clear: 2025-09-06
tags:
  - tag/dp
grasp: 1
difficulty: 1900
optional_difficulty: Easy-Medium
source: https://ac.nowcoder.com/acm/contest/108307/E
article: file:///home/patchouli/Documents/2025NkSummer.ln/10/contest10solution.pdf
---
# 题意
当前有 $n$ 名学生，编号为 $1,2,\dots,n$。第 $i$ 名学生的初始喜爱值为 $a_i$。Sensei 每天可以恰好执行一次如下操作：

- 选取整数 $l, r$ 满足 $1 \le l \le r \le n$，将编号在区间 $[l,r]$ 内的所有学生的喜爱值加 $1$。即对每个整数 $i$（$l \le i \le r$），执行 $a_i := a_i + 1$。

Sensei 希望用最少的天数，令序列 $a_1, a_2, \dots, a_n$ 中 **不同整数的个数不超过 $m$**。求最少需要多少天。
## Input
- 输入包含多组测试数据。
- 第一行是正整数 $T$ $(1 \le T \le 10)$，表示测试用例数。
每个测试用例格式如下：
- 第一行包含两个整数 $n, m$，表示学生数和参数 $m$，满足 $1 \le n \le 400,\ 1 \le m \le 2$。
- 第二行包含 $n$ 个整数 $a_1, a_2, \dots, a_n$，表示每个学生的初始喜爱值，满足 $0 \le a_i \le 100$。
## Output
- 对每个测试用例，输出一行，包含一个非负整数，表示使序列中不同整数个数不超过 $m$ 所需的最少天数。
# Solution
## std_1
若 $m = 1$，直接贪心。注意到全体最大值永远不会变大，每次选取一个极长的最大值没达到全体最大值的段操作即可。  
若 $m = 2$，枚举两种数之差 $\delta$，将数组差分，将操作转化为单点加一减一。dp，维护每个位置之前有几个没配上减一的加一，再维护每个位置的差分是 $0, -\delta, \delta$ 即可。注意 $-\delta, \delta$ 间隔出现。  
复杂度 $O(nV + nV^2)$。
```cpp showLineNumbers title="std_1"
#include <bits/stdc++.h>
using namespace std;

// #define int long long
#define ls               (p << 1)
#define rs               (p << 1 | 1)
#define mid              ((l + r) >> 1)
#define all(_array)      (_array).begin(), (_array).end()
#define msp(_array)      memset(_array, 0x3f, sizeof _array)
#define ms0(_array)      memset(_array, 0, sizeof _array)
#define msn(_array)      memset(_array, -1, sizeof _array)
#define mc(_tar, _array) memcpy(_tar, _array, sizeof _tar)
#define Yes              cout << "Yes" << endl, void()
#define No               cout << "No" << endl, void()
#define YES              cout << "YES" << endl, void()
#define NO               cout << "NO" << endl, void()
#define yes              cout << "yes" << endl, void()
#define no               cout << "no" << endl, void()
#define TAK              cout << "TAK" << endl, void()
#define NIE              cout << "NIE" << endl, void()
#define OK               cerr << "OK" << endl, void()
#define pii              pair<int, int>
#define endl             '\n'

bool                                  bg_memory;
mt19937_64                            rnd(time(0));
chrono::_V2::system_clock::time_point bg_clock, en_clock;
int                                   Case = 1;
const int                             mod  = 1e9 + 7;
const int                             inf  = 1e7;
const int                             bs   = 233;
const double                          eps  = 1e-6;
const int                             N = 400 + 7, M = 200 + 7;

template <class _t1, class _t2>
inline void cmax(_t1 &a, _t2 b)
{
    a = a < b ? b : a;
}

template <class _t1, class _t2>
inline void cmin(_t1 &a, _t2 b)
{
    a = a > b ? b : a;
}

inline int qp(int a, int b, int p = mod)
{
    int res = 1;
    while (b) {
        if (b & 1) res = 1ll * res * a % p;
        a = 1ll * a * a % p;
        b >>= 1;
    }
    return res;
}

inline int sqrt(int x, int r)
{
    int l = 0, ans = 0;
    while (l <= r) {
        if (1ll * mid * mid <= x)
            ans = mid, l = mid + 1;
        else
            r = mid - 1;
    }
    return ans;
}

int n, m, k = 200;
int diff_arr[N], mx;
int ans;
int dp[N][M][2];

void Main()
{

    cin >> n >> m;
    for (int i = 1; i <= n; i++) cin >> diff_arr[i], cmax(mx, diff_arr[i]);
    if (n == 1) return cout << 0 << endl, void();
    ans = 0;
    if (m == 1) {
        int ni = -1;
        for (int i = 1; i <= n; i++) cmax(ni, mx - diff_arr[i]);
        for (int _ = 1; _ <= ni; _++) {
            for (int i = 2; i <= n + 1; i++) {
                if (i == n + 1 || diff_arr[i] == mx) ans += (diff_arr[i - 1] != mx);
            }
            for (int i = 1; i <= n; i++) diff_arr[i] = min(mx, diff_arr[i] + 1);
        }
        cout << ans << endl;
    } else {
        ans = inf;
        for (int i = n; i >= 2; i--) diff_arr[i] = diff_arr[i] - diff_arr[i - 1];
        for (int delta = 0; delta <= k; delta++) {
            msp(dp);
            for (int j = 0; j <= k; j++)
                dp[1][j][0] = dp[1][j][1] =
                    j;  // [i][j][k] i是第几个，j是有多少个没有匹配的-1，0/1表示上一个是-delta或者+delta
            for (int i = 2; i <= n; i++) {
                for (int j = 0; j <= k; j++) {                      // 枚举上一个j
                                                                    // 都是将diff变成0
                    if (diff_arr[i] > 0 && diff_arr[i] - j <= 0) {  // 如果当前>0，且上一个的所有-1数量可以把当前降低到0
                        cmin(dp[i][j - diff_arr[i]][0], dp[i - 1][j][0]);
                        cmin(dp[i][j - diff_arr[i]][1], dp[i - 1][j][1]);
                    }
                    if (diff_arr[i] <= 0) {  // 如果当前<=0，则扩大待匹配的-1数
                        cmin(dp[i][j - diff_arr[i]][0], dp[i - 1][j][0] - diff_arr[i]);
                        cmin(dp[i][j - diff_arr[i]][1], dp[i - 1][j][1] - diff_arr[i]);
                    }
                    if (diff_arr[i] > delta && diff_arr[i] - j <= delta) {  // 如果当前的值>delta，变成delta
                        cmin(dp[i][j - (diff_arr[i] - delta)][0], dp[i - 1][j][1]);
                    }
                    if (diff_arr[i] <= delta) {  // 当前的值<delta，变成delta
                        cmin(dp[i][j - (diff_arr[i] - delta)][0], dp[i - 1][j][1] - (diff_arr[i] - delta));
                    }
                    if (diff_arr[i] > -delta && diff_arr[i] - j <= -delta) {  // 当前的值>-delta
                        cmin(dp[i][j - (diff_arr[i] + delta)][1], dp[i - 1][j][0]);
                    }
                    if (diff_arr[i] <= -delta) {  // 当前的值<=-delta
                        cmin(dp[i][j - (diff_arr[i] + delta)][1], dp[i - 1][j][0] - (diff_arr[i] + delta));
                    }
                }
            }
            for (int j = 0; j <= k; j++) cmin(ans, dp[n][j][0]), cmin(ans, dp[n][j][1]);
            // cout<<d<<" "<<ans<<endl;
        }
        cout << ans << endl;
    }

    return;
}

string RdFile = "woodguy";
bool   en_memory;

signed main()
{
    bg_clock = chrono::high_resolution_clock::now();
    // #ifdef ONLINE_JUDGE
    // freopen((RdFile+".in").c_str(),"r",stdin);
    // freopen((RdFile+".out").c_str(),"w",stdout);
    // #endif
    ios::sync_with_stdio(0);
    cin.tie(0);
    cout.tie(0);
    cin >> Case;
    while (Case--) Main();
    en_clock              = chrono::high_resolution_clock::now();
    auto   duration_clock = chrono::duration_cast<chrono::microseconds>(en_clock - bg_clock);
    double duration_count = duration_clock.count() * 0.001;
    double memory_used    = (&en_memory - &bg_memory) / 1024.0 / 1024;
    // cerr<<"Time:"<<duration_count<<"ms"<<endl;
    // cerr<<"Memory: "<<memory_used<<"MB"<<endl;
    return 0;
}
```

## std_2
注意到将全 $0$ 序列变为给定序列 $x$ 的操作数是
$$
\frac{1}{2}\sum_{i=1}^{n+1} |x_i - x_{i-1}|.
$$
若 $m = 1$，直接贪心。注意到全体最大值永远不会变大，计算将全 $0$ 序列变为序列 $\text{max} - a_i$ 的操作数即可，$\text{max}$ 是 $a_i$ 最大值。  
若 $m = 2$，枚举两种数 $x, y$，利用注意到的公式做 dp。记录当前的 $a_i$ 变为 $x$ 还是 $y$，并维护当前位置初始值与目标值之差与上一个数的初始值与目标值之差的差的绝对值，累加后输出时除以 $2$ 即可。注意枚举的值域上界不止 $100$。  
复杂度 $O(nV + nV^2)$。

#TODO/E
