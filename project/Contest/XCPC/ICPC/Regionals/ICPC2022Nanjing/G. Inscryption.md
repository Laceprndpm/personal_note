---
title: Inscryption
date: 2025-11-04
last_clear: 2025-11-04
tags:
  - QOJ/ICPC/Regionals/Nanjing/2022
  - SUA
  - tag/贪心
grasp: 5
value: 0
difficulty:
optional_difficulty: Easy
source: https://qoj.ac/contest/1093/problem/5420
article:
---
贪心的签到题
```cpp showLineNumbers
void solve()
{
    int n;
    cin >> n;
    vector<int> arr(n + 1);
    for (int i = 1; i <= n; i++) {
        cin >> arr[i];
    }
    auto check = [&](int x) -> optional<pair<int, int>> {
        int cnt = x;
        int num = 1;
        int val = 1;
        for (int i = 1; i <= n; i++) {
            if (arr[i] == 0) {
                if (cnt) {
                    num++;
                    val++;
                    cnt--;
                } else {
                    if (num <= 1) {
                        return nullopt;
                    }
                    num--;
                }
            } else if (arr[i] == 1) {
                num++;
                val++;
            } else if (arr[i] == -1) {
                if (num <= 1) {
                    return nullopt;
                }
                num--;
            }
        }
        return pair<int, int>{val, num};
    };
    int            l = 0, r = n + 1;  // 卡片次数
    pair<int, int> ans;
    while (l <= r) {
        int  mid = (r - l) / 2 + l;
        auto res = check(mid);
        if (res.has_value()) {
            ans = res.value();
            r   = mid - 1;
        } else {
            l = mid + 1;
        }
    }
    if (r == n + 1) {
        cout << -1 << '\n';
        return;
    }
    int gc = gcd(ans.fi, ans.se);
    cout << ans.fi / gc << ' ' << ans.se / gc << '\n';
}
```