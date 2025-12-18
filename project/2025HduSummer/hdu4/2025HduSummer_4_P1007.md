---
title: 咖啡的罪恶
date: 2025-07-28
last_clear: 2025-07-25
tags:
  - tag/打表
grasp: 3
difficulty: 1800
optional_difficulty: Easy-Medium
source: https://acm.hdu.edu.cn/contest/problem?cid=1175&pid=1007
article:
---
# 题意:
定义 `咖啡序列` 为子数组 `arr` 中， `arr[i]` 的值为 `i` 在 `arr` 中出现的次数。(此处 `i` 为 `0-index` )
给定一个 `n` 长度数组 `brr` ，进行 `q` 次查询(以下均为 `1-index` )
- `1` 查询 : 输入 `x p` 修改 `brr[x]` 为 `p`
- `2` 查询 : 输入 `l r` 查询 `l` 到 `r` 最长的 `咖啡序列` 长度。
# 正文:
发现构造起来非常困难，考虑是不是只有固定样式
通过打表可发现，对于不同长度，只有
- $n=4$ : `2 0 2 0` 或 `1 2 1 0`
- $n=5$ : `2 1 2 0 0`
- $n=6$ : 无
- $n\ge7$：$[n-4, \; 2, \; 1, \; \underbrace{0, 0, \dots, 0}_{n-7 \text{ 个}}, \; 1, \; 0, \; 0, \; 0]$
发现非 `0` 元素数量很少，在一个合法的序列里，因此用 `set` 来维护非 `0` 元素下标。注意特判即可
```cpp showLineNubers title="p07.cpp"
array<int, 4> s1{2, 0, 2, 0};
array<int, 4> s2{1, 2, 1, 0};
array<int, 5> s3{2, 1, 2, 0, 0};

void solve()
{
    // 插入所有非0元素
    // 每次修改
    int n, q;
    cin >> n >> q;
    set<int>    index;
    vector<int> arr(n + 1);
    for (int i = 1; i <= n; i++) {
        cin >> arr[i];
    }
    SegmentTree<Info> seg(n + 1);
    for (int i = 1; i <= n; i++) {
        if (arr[i] != 0) {
            index.ins(i);
        }
    }
    index.ins(n + 1);
    auto check_as_head = [&](int x) -> int {
        assert(x >= 1 && x <= n);
        if (arr[x] == 0) return 0;
        i64 ans = 0;
        if (x + 3 <= n) {
            bool ok1 = 1, ok2 = 1;
            for (int i = 0; i < 4; i++) {
                if (arr[x + i] != s1[i]) {
                    ok1 = 0;
                }
                if (arr[x + i] != s2[i]) {
                    ok2 = 0;
                }
            }
            if (ok1 || ok2) {
                ans = max(ans, 4ll);
            }
        }
        if (x + 4 <= n) {
            bool ok = 1;
            for (int i = 0; i < 5; i++) {
                if (arr[x + i] != s3[i]) {
                    ok = 0;
                }
            }
            if (ok) {
                ans = max(ans, 5ll);
            }
        }

        i64 first_val = arr[x];
        i64 len       = first_val + 4;

        if (x + len - 1 <= n && len >= 7) {
            if (arr[x + 1] == 2 && arr[x + 2] == 1 && arr[x + len - 4] == 1) {
                auto it = index.find(x);
                for (int i = 0; i < 4; i++) {
                    if (*it != n + 1) {
                        it = next(it);
                    }
                }
                if (*it - x >= len) {
                    ans = max(ans, len);
                }
            }
        }
        return ans;
    };
    for (int i = 1; i <= n; i++) {
        seg.modify(i, {check_as_head(i), i});
    }
    while (q--) {
        int op;
        cin >> op;
        if (op == 1) {
            int x, p;
            cin >> x >> p;
            int ori_val = arr[x];

            if (ori_val != 0) {
                auto it = index.find(x);
                assert(*it == x);
                index.erase(it);
            }

            if (p != 0) {
                auto res = index.ins(x);
                assert(res.second);
            }

            arr[x]  = p;
            auto it = index.lower_bound(x);
            for (int i = 0; i < 5; i++) {
                if (*it != n + 1) {
                    int res = check_as_head(*it);
                    seg.modify(*it, {res, *it});
                }
                if (it == index.begin()) {
                    break;
                }
                it = prev(it);
            }
            for (int i = 0; i <= 7; i++) {
                int res = check_as_head(x - i);
                seg.modify(x - i, {res, x - i});
                if (x - i <= 1) break;
            }
        } else {
            int l, r;
            cin >> l >> r;
            r++;
            auto [val, pos] = seg.rangeQuery(l, r).x;
            i64 ans         = val;
            if (pos + val - 1 >= r) {
                ans = seg.rangeQuery(l, pos).x.fi;
            }
            cout << ans << '\n';
        }
    }
}
```