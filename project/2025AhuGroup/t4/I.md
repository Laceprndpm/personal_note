---
title: " I. Phony"
date: 2025-09-30
last_clear: 2025-09-30
tags:
  - tag/模拟
  - tag/二分搜索
  - tag/分讨
grasp: 3
value: 5
difficulty:
optional_difficulty: Silver
source: https://qoj.ac/contest/1411/problem/7627
article:
---
```cpp
    // 维护模k下数，用两个deque循环
    // 不行，得用指针
    // 对模k后余数离散化，单独计数
    // vec 5e5
    // 每次得到C，如何更新？
    // 1.每次同时处理下一个数和指针ptr
    // 2.根据当前指针位置，确定是否要在指针移到末尾循环前吃掉这个数
    // 不，是计算出吃掉下一个数需要bp到多少，随后开始吃，并且移动指针
```
逻辑难以设计/实现，最终采取的实现是，每次将指针移动到开头后，一次性全部添加，后来每次移动指针，都要尝试添加
有点异步更新的味道
常见坑点：
- 忘记指针的开闭，即此处 `fkptr` 指针是，\[0, fkptr\)，和\[fkptr, n\)，均为左闭右开
- `Fenwick` 的 `sum` 命令是开区间
- `Fenwick` 的 `select` 命令是找到第一个 `sum(x)>k`
- 正因如此，所以 $t = T - 1$ 后，用 `select(t)` 才能找第 `t` 位

```cpp showLineNumbers
#include <algorithm>
#include <cassert>
#include <deque>
#include <functional>
#include <ios>
#include <iostream>
#include <numeric>
#include <utility>
#include <vector>

using namespace std;
using i64 = long long;
using u64 = uint64_t;

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

template <typename T>
T floor(T a, T b)
{
    return a / b - (a % b && (a ^ b) < 0);
}

template <typename T>
T ceil(T x, T y)
{
    return floor(x + y - 1, y);
}

template <typename T>
T bmod(T x, T y)
{
    return x - y * floor(x, y);
}

template <typename T>
pair<T, T> divmod(T x, T y)
{
    T q = floor(x, y);
    return {q, x - q * y};
}

template <typename T, typename U>
T SUM(const U& A)
{
    return std::accumulate(A.begin(), A.end(), T{});
}

template <typename T>
struct Fenwick {
    int            n;
    std::vector<T> a;

    Fenwick(int n_ = 0) { init(n_); }

    void init(int n_)
    {
        n = n_;
        a.assign(n, T{});
    }

    void add(int x, const T& v)
    {
        for (int i = x + 1; i <= n; i += i & -i) {
            a[i - 1] = a[i - 1] + v;
        }
    }

    // 开区间
    T sum(int x)
    {
        T ans{};
        for (int i = x; i > 0; i -= i & -i) {
            ans = ans + a[i - 1];
        }
        return ans;
    }

    // 左闭右开
    T rangeSum(int l, int r) { return sum(r) - sum(l); }

    int select(const T& k)
    {
        int x = 0;
        T   cur{};
        for (int i = 1 << std::__lg(n); i; i /= 2) {
            if (x + i <= n && cur + a[x + i - 1] <= k) {
                x += i;
                cur = cur + a[x - 1];
            }
        }
        return x;
    }
};

template <class T>
struct RangeFenwick {
    int        n;
    Fenwick<T> d1, d2;

    RangeFenwick(int n_) : n(n_), d1(n_), d2(n_) {}

    void add(i64 k, T v)
    {
        T v1 = k * v;
        d1.add(k, v), d2.add(k, v1);
        // 在diff的k位置+v
    }

    void add(i64 l, i64 r, T v)
    {
        add(l, v), add(r, -v);  // 将区间加差分为两个前缀加
    }

    T getsum(i64 x) { return x * d1.sum(x) - d2.sum(x); }

    T getsum(i64 l, i64 r) { return getsum(r) - getsum(l); }
};

void solve()
{
    i64 n, m;
    cin >> n >> m;
    i64 k;
    cin >> k;
    vector<i64> ai(n);
    for (int i = 0; i < n; i++) {
        cin >> ai[i];
    }
    Fenwick<int> fk(n);
    sort(all(ai), greater<>());
    int aiptr = 0;
    int fkptr = 0;  // 正在哪个数上
    i64 cstep = floor(ai[0], k);

    auto update = [&](void) {
        if (fkptr >= n) {
            fkptr -= n;
            cstep--;
        }
    };
    // 维护模k下数，用两个deque循环
    // 不行，得用指针
    // 对模k后余数离散化，单独计数
    // vec 5e5
    // 每次得到C，如何更新？
    // 1.每次同时处理下一个数和指针ptr
    // 2.根据当前指针位置，确定是否要在指针移到末尾循环前吃掉这个数
    // 不，是计算出吃掉下一个数需要bp到多少，随后开始吃，并且移动指针
    vector<i64> lihua = ai;
    for (i64& i : lihua) {
        i = bmod(i, k);
    }
    sort(all(lihua), greater<>());
    vector<int> offsetvec(n);
    vector<int> cnt(n);
    vector<int> belongto(n);
    for (int i = 0; i < n; i++) {
        i64  val    = bmod(ai[i], k);
        auto it     = lower_bound(all(lihua), val, greater<>());
        int  pos    = it - lihua.begin();
        belongto[i] = pos + cnt[pos]++;
    }

    auto eatone = [&](int idx) noexcept {
        pair<i64, i64> step_ptr;
        i64            val  = ai[idx];
        i64            step = floor(val, k);
        step_ptr            = {step, belongto[idx]};
        return step_ptr;
    };

    auto gogo = [&](u64 cost) {
        assert(cost);
        const int toend = fk.rangeSum(fkptr, n);
        if (cost <= toend) {
            int nfkptr = fk.select(fk.sum(fkptr) + cost);
            fkptr      = nfkptr;
            update();
            return;
        }
        cost -= toend;
        fkptr = n;
        update();
        const i64 curnum  = fk.rangeSum(0, n);
        const i64 stepcnt = curnum ? cost / curnum : 0;
        cstep -= stepcnt;
        cost -= stepcnt * curnum;
        if (cost) {
            const int nfkptr = fk.select(cost);
            fkptr            = nfkptr;
            update();
        } else {
            fkptr = 0;
        }
        assert(fkptr < n);
    };
    auto go = [&](i64 c) {
        while (c) {
            if (aiptr == n) {
                gogo(c);
                c = 0;
                return;
            }
            auto const [nstep, nfkptr] = eatone(aiptr);

            i64 need = (cstep - nstep - 1) * fk.rangeSum(0, n) + fk.rangeSum(fkptr, n);
            assert(need >= 0);
            if (need == 0) {
                cstep = nstep, fkptr = 0;
            } else {
                gogo(min(need, c));
            }
            update();
            c -= min(need, c);
            while (aiptr < n && eatone(aiptr).first == cstep) {
                fk.add(belongto[aiptr], 1);
                aiptr++;
            }
            while (aiptr < n && eatone(aiptr).first == cstep - 1 && eatone(aiptr).second < fkptr) {
                fk.add(belongto[aiptr], 1);
                aiptr++;
            }
        }
    };
    while (m--) {
        char cmd;
        cin >> cmd;
        i64 t;
        cin >> t;
        if (cmd == 'A') {
            t--;
            update();
            i64 suf = fk.rangeSum(fkptr, n);
            if (t < suf) {
                cout << lihua[fk.select(t + fk.sum(fkptr))] + cstep * k << '\n';
            } else {
                t -= suf;
                i64 pre = fk.rangeSum(0, fkptr);
                if (t < pre) {
                    cout << lihua[fk.select(t)] + (cstep - 1ll) * k << '\n';
                } else {
                    t -= pre;
                    cout << ai[aiptr + t] << '\n';
                }
            }
        } else {
            go(t);
        }
    }
}

signed main(signed argc, char** argv)
{
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    int t = 1;
    while (t--) {
        solve();
    }
    return 0;
}

```