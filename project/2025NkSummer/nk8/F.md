---
title: Broken LED Lights
date: 2025-09-06
last_clear: 2025-09-06
tags:
  - tag/thinking/划分
  - NK
  - tag/bitmasks
grasp: 3
difficulty:
optional_difficulty: Medium
source: https://ac.nowcoder.com/acm/contest/108305/F
article:
---
# 题意
给定数位 `0` 到 `9` 的 7 根 LED 灯管表示。考虑有 `n` 个长度为 `m` 的数字（每个数字由 `m` 个十进制数位组成），我们可以选择保留其中的一些灯管（在所有位上保留相同位置的若干根灯管），使得仅用这些保留的灯管即可区分这 `n` 个数。
区分的定义：对于任意两个不同的数，若在剩下的灯管中存在至少一根灯管，其在这两个数对应位置的点亮/熄灭状态不同，则这两个数可被区分。
输出要求：输出最少需要保留的灯管根数，使得所有 `n` 个数两两可区分。
约束条件：
- 多组数据；
- $1 \le n \le 100$；
- $1 \le m \le 3$。
## Input
输入包含多组测试数据。  
第一行给出一个正整数 $T$ $(1 \le T \le 10000)$，表示测试数据的组数。
每组测试数据的描述如下：
- 第一行包含两个整数 $n, k$ $(3 \le n \le 2\times 10^5,\ 1 \le k \le 10^9)$。  
- 第二行包含 $n$ 个整数 $a_1, a_2, \dots, a_n$ $(1 \le a_i \le 10^9)$。
保证所有的 $a_i$ 两两不同，且所有测试数据的 $n$ 之和不超过 $2\times 10^5$。
## Output
对于每个测试用例，输出一个整数，表示需要保持运行的最少灯管数量。
# Solution
## my（劣解）
枚举保留多少个灯管后，检查亮起后有没有完全相同的即可，难在卡常。
首先，虽然范围只有 `1-15` 但是依然用二分来省去几个常数。
其次，虽然 $n\le 100$ 但需要用桶排而不是 sort 来省去一个 `log100` 。

```cpp showLineNumbers title="lace"
int digits[10] = {
    0b1111110,  // 0
    0b0110000,  // 1
    0b1101101,  // 2
    0b1111001,  // 3
    0b0110011,  // 4
    0b1011011,  // 5
    0b1011111,  // 6
    0b1110000,  // 7
    0b1111111,  // 8
    0b1111011   // 9
};

struct Init {
    array<vector<int>, 15> vec;

    Init()
    {
        for (int i = 0; i < (1 << 21) - 1; i++) {
            if (__builtin_popcount(i) > 14) continue;
            vec[__builtin_popcount(i)].push_back(i);
        }
        for (int i = 0; i < 15; i++) {
            sor(vec[i]);
        }
    }
} init;

int vis[1 << 21];
u32 tag = 0;

void solve()
{
    int n, m;
    cin >> n >> m;
    vector<int> arr(n);
    for (int i = 0; i < n; i++) {
        int x;
        cin >> x;
        int val = 0;
        for (int j = 0; j < m; j++) {
            val <<= 7;
            val |= digits[(x % 10)];
            x /= 10;
        }
        arr[i] = val;
    }
    if (n == 1) {
        cout << 0 << '\n';
        return;
    }
    vector<int>         brr(n);
    function<bool(int)> check = [&](int x) -> bool {
        tag++;
        if (tag == 0) {
            memset(vis, 0, sizeof(vis));
        }
        for (int i = 0; i < n; i++) {
            if (vis[x & arr[i]] == tag) return 0;
            vis[x & arr[i]] = tag;
        }

        return 1;
    };
    function<bool(int)> che = [&](int num) -> bool {
        if (num == 15) return true;
        for (int i = 0; i < init.vec[num].size(); i++) {
            if (check(init.vec[num][i])) {
                return 1;
            }
        }
        return 0;
    };
    int l = 1, r = 15;
    int ans = 0;
    while (l <= r) {
        int mid = (r + l) / 2;
        if (che(mid)) {
            r   = mid - 1;
            ans = mid;
        } else {
            l = mid + 1;
        }
    }
    cout << ans << '\n';
}
```
## std
最暴力做法是搜索用哪些灯管后检查 n 个数不同，时间复杂度
$O(2^{7m}n)$，复杂度跑满是过不了的，加一点最优性剪枝就可以过。
但不要乱加剪枝，例如在没有优化分支数量时，每一层增加 $O(n)$ 或
$O(7m)$ 的代价去找更优分支就会被卡。

进一步发现主要是剪枝分析代价过高，可以每个位置的 $2^7$ 种候选整体
分析，有一定效果但是不大。

注意到如果只有一个位置，那么最多用 5 个灯管即可，以及 $2^7$ 很多重
复情况没必要，具体来说：
• 如果两种候选方式对任意数位集合的等价类划分结果一样，那么只需要
  留一个使用灯管数最小的；
• 如果一种候选方式能比另一种候选方式划分得更细（等价类不相交或者
  包含），那么另一种候选方式留下的前提是它使用灯管数严格小于该候
  选方式。

缩一下只有 58 种情况，时间复杂度降低至 $O(58^mn)$ 就可以过了。
**PS:这里实际上展示了61种情况，相同划分的情况被重复计算了，如果有必要你可以从后往前跑一边，或者修改判断条件，无论如何，预处理还是$O(n^2)$**。
```cpp showLineNumbers title="F_0_p0"
int digits[10] = {
    0b1111110,  // 0
    0b0110000,  // 1
    0b1101101,  // 2
    0b1111001,  // 3
    0b0110011,  // 4
    0b1011011,  // 5
    0b1011111,  // 6
    0b1110000,  // 7
    0b1111111,  // 8
    0b1111011   // 9
};

bool is_subof(vector<int> const& dest, vector<int> const& src)
{
    int n = src.size();
    assert(n == dest.size());
    map<int, int> tmp;
    for (int i = 0; i < n; i++) {
        if (!tmp.contains(src[i])) {
            tmp[src[i]] = dest[i];
        } else {
            if (tmp[src[i]] != dest[i]) {
                return 0;
            }
        }
    }
    return 1;
}

void solve()
{
    vector<vector<int>> allhua(1 << 7);
    for (int i = 0; i < (1 << 7); i++) {
        vector<int> tmphua(10);
        for (int j = 0; j <= 9; j++) {
            tmphua[j] = digits[j] & i;
        }
        allhua[i] = tmphua;
    }
    vector<int> ans;
    for (int i = 0; i < (1 << 7); i++) {
        bool ok = 1;
        cout << i << '\n';
        for (int j = 0; j < (1 << 7); j++) {
            if (j == i) continue;
            if (is_subof(allhua[i], allhua[j]) && !is_subof(allhua[j], allhua[i])
                && __builtin_popcount(j) <= __builtin_popcount(i)) {
                ok = 0;
                break;
            }
        }
        if (ok) {
            ans.pb(i);
        }
    }
    cout << ans.size() << '\n';
    for (int i : ans) {
        cout << i << ',';
    }
}
```

```cpp showLineNumbers title="F_0_p1"
int digits[10] = {
    0b1111110,  // 0
    0b0110000,  // 1
    0b1101101,  // 2
    0b1111001,  // 3
    0b0110011,  // 4
    0b1011011,  // 5
    0b1011111,  // 6
    0b1110000,  // 7
    0b1111111,  // 8
    0b1111011   // 9
};
int ARR[] = {0,  1,  2,  3,  4,  5,  6,  7,  8,   9,   10,  12,  14,  16,  17,  24,  25,  32,  33, 34, 35,
             36, 37, 38, 39, 40, 41, 42, 44, 46,  48,  49,  56,  57,  64,  65,  66,  67,  68,  69, 70, 71,
             72, 80, 81, 88, 96, 97, 98, 99, 100, 101, 102, 103, 104, 111, 112, 113, 119, 120, 127};

struct Init {
    array<vector<int>, 15> vec;

    Init()
    {
        // for (int i = 0; i < (1 << 21) - 1; i++) {
        //     if (__builtin_popcount(i) > 14) continue;
        //     vec[__builtin_popcount(i)].push_back(i);
        // }
        // for (int i = 0; i < 15; i++) {
        //     sor(vec[i]);
        // }
        for (int i = 0; i < 61; i++) {
            for (int j = 0; j < 61; j++) {
                for (int k = 0; k < 61; k++) {
                    int cur = (ARR[i] << 14) | (ARR[j] << 7) | (ARR[k]);
                    if (__builtin_popcount(cur) > 14) continue;
                    vec[__builtin_popcount(cur)].push_back(cur);
                }
            }
        }
    }
} init;

int vis[1 << 21];
u32 tag = 0;

void solve()
{
    int n, m;
    cin >> n >> m;
    vector<int> arr(n);
    for (int i = 0; i < n; i++) {
        int x;
        cin >> x;
        int val = 0;
        for (int j = 0; j < m; j++) {
            val <<= 7;
            val |= digits[(x % 10)];
            x /= 10;
        }
        arr[i] = val;
    }
    if (n == 1) {
        cout << 0 << '\n';
        return;
    }
    vector<int>         brr(n);
    function<bool(int)> check = [&](int x) -> bool {
        tag++;
        if (tag == 0) {
            memset(vis, 0, sizeof(vis));
        }
        for (int i = 0; i < n; i++) {
            if (vis[x & arr[i]] == tag) return 0;
            vis[x & arr[i]] = tag;
        }

        return 1;
    };
    function<bool(int)> che = [&](int num) -> bool {
        if (num == 15) return true;
        for (int i = 0; i < init.vec[num].size(); i++) {
            if (check(init.vec[num][i])) {
                return 1;
            }
        }
        return 0;
    };
    int l = 1, r = 15;
    int ans = 0;
    while (l <= r) {
        int mid = (r + l) / 2;
        if (che(mid)) {
            r   = mid - 1;
            ans = mid;
        } else {
            l = mid + 1;
        }
    }
    cout << ans << '\n';
}
```