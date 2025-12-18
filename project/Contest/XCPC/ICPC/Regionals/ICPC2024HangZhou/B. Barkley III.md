---
title: Barkley III
date: 2025-10-28
last_clear: 2025-10-28
tags:
  - QOJ/ICPC/Regionals/Hangzhou/2024
  - SUA
  - tag/数据结构/线段树
  - tag/bitmasks
  - tag/etc/卡常
grasp: 3
value: 3
difficulty:
optional_difficulty: Easy-Medium
source: https://qoj.ac/contest/1893/problem/9727
article: https://qoj.ac/contest/1893/problem/9727
---
该题卡常严重，哪怕多查询一次都可能导致TLE，因此线段树需要尽可能减少查询次数，可以将二分线段树过程和求答案过程合并来加速。
千万注意不要写成双log，比如如果在每层都调用一次range_Query就会导致双log无法通过
以及这个查询方法可以总结一下：每个节点只有左面的节点会被查询后进入，所以可以在return -1时，统计答案，如果左面已经进入了，则直接调用一次rangeQuery(m, r)，就可以避免双log。
当然，这题最核心的还是想出来用loss变量来维护那个只有一次没出现的值是谁，避免64的大常数
```cpp showLineNumbers
    template <class F>
    int myFind(int p, int l, int r, int x, int y, F pred)
    {
        if (l >= y || r <= x) {
            return -1;
        }
        if (x <= l && r <= y && !pred(info[p])) {
            ans &= info[p].x;
            return -1;
        }
        if (r - l == 1) {
            return l;
        }
        int m = (l + r) / 2;
        push(p);
        int res = myFind(2 * p, l, m, x, y, pred);
        if (res == -1) {
            res = myFind(2 * p + 1, m, r, x, y, pred);
        } else {
            ans &= rangeQuery(2 * p + 1, m, r, x, y).x;
        }
        return res;
    }

    template <class F>
    int myFind(int l, int r, F pred)
    {
        return myFind(1, 0, n, l, r, pred);
    }

constexpr u64 B = (1ull << 63) - 1;

struct Tag {
    u64 x = B;

    void apply(Tag t) { x &= t.x; }
};

struct Info {
    u64 x    = B;
    u64 loss = 0;

    // a & b = 0
    // a | b = 1
    // a ^ b = 1
    void apply(Tag t)
    {
        x &= t.x;
        if (!(loss >> 63 & 1))
            loss &= t.x;
        else
            loss = x ^ B | 1ull << 63;
    }
};

Info operator+(Info a, Info b)
{
    Info c;
    c.x    = a.x & b.x;
    c.loss = (a.loss & b.x) | (b.loss & a.x);
    return c;
}


void solve()
{
    int n, q;
    cin >> n >> q;
    vector<Info> arr(n);
    for (int i = 0; i < n; i++) {
        cin >> arr[i].x;
        arr[i].loss = (arr[i].x ^ B) | (1ull << 63);
    }
    LazySegmentTree<Info, Tag> lzt(std::move(arr));
    u64                        found = 0;

    auto pred = [&found](const Info &x) -> bool { return x.loss & found; };
    while (q--) {
        int op;
        cin >> op;
        switch (op) {
            case 1: {
                int l, r;
                cin >> l >> r;
                l--;
                u64 x;
                cin >> x;
                lzt.rangeApply(l, r, {x});
                break;
            }
            case 2: {
                int s;
                cin >> s;
                u64 x;
                cin >> x;
                lzt.modify(s - 1, {x, (x ^ B) | 1ull << 63});
                break;
            }
            case 3: {
                int l, r;
                cin >> l >> r;
                l--;
                auto res    = lzt.rangeQuery(l, r);
                u64  target = res.loss;
                for (int i = 63; i >= 0; i--) {
                    if (target >> i & 1ull) {
                        found = 1ull << i;
                        break;
                    }
                }
                if (found) {
                    ans     = B;
                    int mid = lzt.myFind(l, r, pred);
                    cout << ans << '\n';
                } else {
                    cout << res.x << '\n';
                }
                found = 0;
            }
        }
    }
    cout << '\n';
}
```
