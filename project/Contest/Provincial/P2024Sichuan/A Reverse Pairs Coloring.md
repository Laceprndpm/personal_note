---
title: Reverse Pairs Coloring
date: 2025-10-19
last_clear: 2025-10-20
tags:
  - QOJ/ICPC/Regionals/Asia_East_Continent/Sichuan/2024
  - tag/数据结构/树状数组
  - tag/dsu
  - tag/thinking/reverse
  - WHO/UESTC
grasp: 4
value: 4
difficulty:
optional_difficulty: Easy-Medium
source: https://qoj.ac/contest/1862/problem/9663
article: https://qoj.ac/download.php?type=attachments&id=1862&r=2
---
# Solution
注意到从后往前考虑，每次会清空同列下面和同行右面，可以转化成当到这里才把自己假如，然后写代码
```cpp showLineNumbers
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

    T sum(int x)
    {
        T ans{};
        for (int i = x; i > 0; i -= i & -i) {
            ans = ans + a[i - 1];
        }
        return ans;
    }

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

// 9
// 9 7 5 3 1 8 6 4 2
// |######## |
// |######   |
// |####     |
// |##       |
// |         |
// | # # #   |
// | # #     |
// | #       |
// |         |
void solve()
{
    int n;
    cin >> n;
    vector<int> arr(n + 1);
    for (int i = 1; i <= n; i++) {
        cin >> arr[i];
    }
    Fenwick<int> fk(n + 1);
    reverse(arr.begin() + 1, arr.end());
    int         last = n + 1;
    i64         ans  = 0;
    vector<int> alreadyin(n + 1);
    arr.pb(0);
    for (int i = 1; i <= n + 1; i++) {
        int cur = arr[i];
        ans += max(fk.rangeSum(cur, last), 0);
        if (last != n + 1) {
            if (last + 1 <= n && alreadyin[last + 1]) {
                fk.add(last + 1, -1);
            }
            if (last - 1 < 1 || !alreadyin[last - 1]) {
                fk.add(last, 1);
            }
            alreadyin[last] = 1;
        }
        last = cur;
    }
    cout << ans << '\n';
}
```
