---
title: Chat Program
date: 2025-09-10
last_clear: 2025-11-04
tags:
  - QOJ/ICPC/Regionals/Nanjing/2022
  - SUA
  - tag/贪心
  - tag/二分搜索
  - tag/滑动窗口
grasp: 4
value: 2
difficulty:
optional_difficulty: Easy-Medium
source: https://codeforces.com/gym/104128/problem/D
article:
---
巨丑，但是能过，且思路没有大问题，似乎多带个 $\log$ ，但是不想改了
二分第k大，然后就是一堆01，每一个0值可以逆向出一个区间，前缀和后查询有没有位置可以将足够的0变成1即可
```cpp
void solve()
{
    int n, k, m, c, d;
    cin >> n >> k >> m >> c >> d;
    k = n - k + 1;
    vector<int> arr(n);
    for (int i = 0; i < n; i++) {
        cin >> arr[i];
    }
    i64          l = 0, r = 3e14;
    vector<Info> base(n, {n});

    auto find_r = [&](i64 i, i64 need) -> i64 {
        i64 l = 0, r = n;
        i64 res = -1;
        while (l <= r) {
            i64 mid = (r - l) / 2 + l;
            if (1ll * (i - mid) * d + c >= need) {
                res = mid;
                l   = mid + 1;
            } else {
                r = mid - 1;
            }
        }
        return res;
    };
    auto check = [&](i64 x) {
        LazySegmentTree<Info, Tag> lzt(base);
        i64                        bias = 0;
        for (int i = 0; i < n; i++) {
            i64 need = max(0ll, x - arr[i]);
            if (need == 0) {
                bias++;
                continue;
            }
            if (need > 1ll * max((m - 1), i) * d + c) {
                continue;
            };
            // [x, x + m - 1], x + m - 1 >= i
            // c + (i - x) * d >= need
            // (i - x) * d >= need - c
            // (i - x) >= (need - c) / d
            // x <= i - (need - c) / d
            int l = i + 1 - m;
            int r;
            if (d) r = find_r(i, need);
            if (need <= c) {
                r = i;
            }
            l = max(0, l);
            r = min(n - 1, r);
            lzt.rangeApply(l, r + 1, {-1});
        }
        return lzt.rangeQuery(0, n).x <= k - 1 + bias;
    };
    i64 ans = 0;
    while (l <= r) {
        i64 mid = (r - l) / 2 + l;
        if (check(mid)) {
            ans = mid;
            l   = mid + 1;
        } else {
            r = mid - 1;
        }
    }
    cout << ans << '\n';
}
```