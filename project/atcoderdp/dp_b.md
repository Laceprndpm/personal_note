---
title: B - Frog 2
date: 2025-07-06
last_clear: 2025-06-30
tags: 
grasp: 3
difficulty: 
source: https://atcoder.jp/contests/dp/tasks/dp_b
---
## Solution
### Easy
- $2 \le N \le 10^5$
- $1 \le K \le 100$
- $1 \le h_i \le 10^4$
每只青蛙只能从前 $K$ 只转移，不难写出转移方程
$$
\text{dp[i]} = \min_{j = \max(i - k, 1)}^{j < i} \text{dp[j]} + |h_i - h_j|
$$
暴力转移时，每只青蛙最多考虑前 $K$ 只青蛙，复杂度为 $O(NK)$
而本题中 $K \le 100$ ，该复杂度可轻松通过
```cpp showLineNumbers title="dp_b_teach_0.cpp"
#include <algorithm>
#include <iostream>
#include <vector>
using namespace std;
using i64 = long long;

constexpr int INF = 1e9;

int main()
{
    int n, k;
    cin >> n >> k;
    vector<i64> hi(n + 1);  // 读入每个点的高度
    for (int i = 1; i <= n; i++) {
        cin >> hi[i];
    }
    vector<i64> dp(n + 1, INF);  // 初始化为无穷大
    dp[1] = 0;
    for (int i = 2; i <= n; i++) {
        for (int j = max(i - k, 1); j < i; j++) {  // 对每个点，向后转移
            dp[i] = min(dp[i], dp[j] + abs(hi[i] - hi[j]));
        }
    }
    cout << dp[n] << '\n';
    return 0;
}
```
### Hard
- $2 \le N \le 10^5$
- $1 \le K \le 10^5$
- $1 \le h_i \le 10^4$
注意到可以将绝对值拆开后分别维护 $h_j \le h_i$ 和 $h_j > h_i$ 的情况，因此可以用线段树优化dp。因为每个高度可能出现多个点，因此erase时需要一个 `vector<multiset<int>>` 来辅助修改。详情查看代码，复杂度 $O(n \log 2e4)$ 其中 $1e4 = \max(h_i)$ ，开 $2e4$ 保险一点。
这里不要被 `SegmentTree<T>` 绕晕啦，就把他当作一个抽象数据结构，支持$O(\log n)$ 单点修改和 $O(\log n)$ 区间查询最小值即可。你现在可以视自己情况，直接跳过这一段。
```cpp showLineNumbers title="dp_b_teach_1.cpp"
#include <algorithm>
#include <functional>
#include <iostream>
#include <set>
#include <vector>
using namespace std;
using i64 = long long;

constexpr int INF = 1e9;

template <class Info>
struct SegmentTree {
    int          n;
    vector<Info> info;

    SegmentTree() : n(0) {}

    SegmentTree(int n_, Info v_ = Info()) { init(n_, v_); }

    template <class T>
    SegmentTree(vector<T> init_)
    {
        init(init_);
    }

    void init(int n_, Info v_ = Info()) { init(vector(n_, v_)); }

    template <class T>
    void init(vector<T> init_)
    {
        n = init_.size();
        info.assign(4 << __lg(n), Info());
        function<void(int, int, int)> build = [&](int p, int l, int r) {
            if (r - l == 1) {
                info[p] = init_[l];
                return;
            }
            int m = (l + r) / 2;
            build(2 * p, l, m);
            build(2 * p + 1, m, r);
            pull(p);
        };
        build(1, 0, n);
    }

    void pull(int p) { info[p] = info[2 * p] + info[2 * p + 1]; }

    void modify(int p, int l, int r, int x, const Info &v)
    {
        if (r - l == 1) {
            info[p] = v;
            return;
        }
        int m = (l + r) / 2;
        if (x < m) {
            modify(2 * p, l, m, x, v);
        } else {
            modify(2 * p + 1, m, r, x, v);
        }
        pull(p);
    }

    void modify(int p, const Info &v) { modify(1, 0, n, p, v); }

    Info rangeQuery(int p, int l, int r, int x, int y)
    {
        if (l >= y || r <= x) {
            return Info();
        }
        if (l >= x && r <= y) {
            return info[p];
        }
        int m = (l + r) / 2;
        return rangeQuery(2 * p, l, m, x, y) + rangeQuery(2 * p + 1, m, r, x, y);
    }

    Info rangeQuery(int l, int r) { return rangeQuery(1, 0, n, l, r); }

    template <class F>
    int findFirst(int p, int l, int r, int x, int y, F &&pred)
    {
        if (l >= y || r <= x) {
            return -1;
        }
        if (l >= x && r <= y && !pred(info[p])) {
            return -1;
        }
        if (r - l == 1) {
            return l;
        }
        int m   = (l + r) / 2;
        int res = findFirst(2 * p, l, m, x, y, pred);
        if (res == -1) {
            res = findFirst(2 * p + 1, m, r, x, y, pred);
        }
        return res;
    }

    template <class F>
    int findFirst(int l, int r, F &&pred)
    {
        return findFirst(1, 0, n, l, r, pred);
    }

    template <class F>
    int findLast(int p, int l, int r, int x, int y, F &&pred)
    {
        if (l >= y || r <= x) {
            return -1;
        }
        if (l >= x && r <= y && !pred(info[p])) {
            return -1;
        }
        if (r - l == 1) {
            return l;
        }
        int m   = (l + r) / 2;
        int res = findLast(2 * p + 1, m, r, x, y, pred);
        if (res == -1) {
            res = findLast(2 * p, l, m, x, y, pred);
        }
        return res;
    }

    template <class F>
    int findLast(int l, int r, F &&pred)
    {
        return findLast(1, 0, n, l, r, pred);
    }
};

struct Info {
    int x;

    Info() : x(INF) {}

    Info(int _x) : x(_x) {}
};

Info operator+(const Info &a, const Info &b) { return {min(a.x, b.x)}; }

int main()
{
    int n, k;
    cin >> n >> k;
    vector<int> hi(n + 1);  // 读入每个点的高度
    for (int i = 1; i <= n; i++) {
        cin >> hi[i];
    }
    // 当前在i点，如果从j点跳过来
    SegmentTree<Info> seg_subtract(2e4);
    // 假设h_i > h_j，则dp[i] = dp[j] + |h_i - h_j| = (dp[j] - h_j) + h_i
    SegmentTree<Info> seg_add(2e4);
    // 即h_i <= h_j，的情况，同理，dp[i] = dp[j] + |h_i - h_j| = (dp[j] + h_j) - h_i
    //
    vector<multiset<int>> multisets(2e4);
    vector<int>           dp(n + 1, INF);  // 初始化为无穷大
    auto                  update = [&](int i) {
        multisets[hi[i]].insert(dp[i]);
        seg_subtract.modify(hi[i], Info{*multisets[hi[i]].begin() - hi[i]});
        seg_add.modify(hi[i], Info{*multisets[hi[i]].begin() + hi[i]});
    };

    auto remove = [&](int i) {
        multisets[hi[i]].extract(dp[i]);
        if (multisets[hi[i]].empty()) {
            seg_subtract.modify(hi[i], Info{});
            seg_add.modify(hi[i], Info{});
            return;
        }
        seg_subtract.modify(hi[i], Info{*multisets[hi[i]].begin() - hi[i]});
        seg_add.modify(hi[i], Info{*multisets[hi[i]].begin() + hi[i]});
    };
    dp[1] = 0;
    update(1);
    for (int i = 2; i <= n; i++) {
        if (i - k - 1 >= 1) {
            remove(i - k - 1);
        }
        dp[i] = min(dp[i], seg_add.rangeQuery(hi[i], 2e4).x - hi[i]);
        dp[i] = min(dp[i], seg_subtract.rangeQuery(0, hi[i]).x + hi[i]);
        update(i);
    }
    cout << dp[n] << '\n';
    return 0;
}
```
备注：C++ 模板在最简单的情况下，就是在编译时由编译器自动把你写的“泛型代码”按需要的类型生成一份等价的代码，相当于**类型安全的宏替换**。
后续你可以选择学习使用了C++模板 的板子[^板子]，个人相当推荐，有以下理由
1. 抽象能力强，减少重复代码。核心逻辑只写一次，`Info` 怎么定义完全交给用户。比赛里常常要切换最小值/最大值/区间和等各种功能，不用复制粘贴一份线段树，而是换一个 `Info` 结构就能重用。
2. 零运行时开销。模板是编译期展开的，不像虚函数要多一层动态派发。写 `vector<int>` 和 `vector<long long>` 并不会在性能上额外花钱
3. 灵活组合，快速定制。模板可以嵌套
4. 方便调试逻辑。模板使用熟练后可以快速定位核心逻辑，抄模板上去再改接口或者某些细节相比于边回忆边写还根据题目适应，出错的概率偏小（也许？）且ACM中不涉及复杂的模板替换，一般来说只有简单类型嵌套，不需要过于担心编译错误

[^板子]: 即常用的可复用代码模板，线段树、重链剖分等，大致逻辑类似即可做成板子以复用。
