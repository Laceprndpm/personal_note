---
title: Paimon Sorting
date: 2025-10-30
last_clear: 2025-10-30
tags:
  - QOJ/ICPC/Regionals/Nanjing/2021
  - SUA
  - tag/Fenwick
  - tag/打表
grasp: 3
value: 5
difficulty:
optional_difficulty: Easy-Medium
source: https://qoj.ac/contest/1236/problem/6434
article: https://zhuanlan.zhihu.com/p/441174109
---
注意点：
1. 第一次特殊
2. 想清楚再写
```cpp showLineNumbers title="打表"
void solve()
{
    int n;
    cin >> n;
    vector<int> arr(n + 1);
    for (int i = 1; i <= n; i++)
    {
        cin >> arr[i];
    }
    cout << "cnt : ";
    for (int k = 1; k <= n; k++)
    {
        vector<int> brr = vector<int>(arr.begin(), arr.begin() + 1 + k);
        int cnt = 0;
        for (int ii = 1; ii <= k; ii++)
        {
            for (int jj = 1; jj <= k; jj++)
            {
                if (brr[ii] < brr[jj])
                {
                    cnt++;
                    swap(brr[ii], brr[jj]);
                }
            }
        }
        cout << cnt << ' ';
    }
    cout << '\n';
    for (int ii = 1; ii <= n; ii++)
    {
        for (int jj = 1; jj <= n; jj++)
        {
            if (arr[ii] < arr[jj])
            {
                swap(arr[ii], arr[jj]);
            }
        }
        cout << "i = " << ii << " : ";
        for (int jj = 1; jj <= n; jj++)
        {
            cout << arr[jj] << ' ';
        }
        cout << '\n';
    }
    // 1
    // 10
    // 10 9 8 7 6 5 4 3 2 1
    // 0 1 3 6 10 15 21 28 36 45
    // 1 2 3 4 5 6 7 8 9 10

    // 1
    // 10
    // 1 2 3 4 5 6 7 8 9 10
    // 0 2 4 6 8 10 12 14 16 18
    // 1 2 3 4 5 6 7 8 9 10
    cout << '\n';
    for (int i = 1; i <= n; i++)
    {
        cout << arr[i] << ' ';
    }
    cout << '\n';
}
```
解题：
```cpp showLineNumbers
#include <bits/stdc++.h>
using namespace std;
using i64 = long long;
#define pb push_back
#define bg(x) begin(x)
#define all(x) bg(x), end(x)
#define int long long
template <typename T>
struct Fenwick
{
    int n;
    std::vector<T> a;

    Fenwick(int n_ = 0)
    {
        init(n_);
    }

    void init(int n_)
    {
        n = n_;
        a.assign(n, T{});
    }

    void add(int x, const T &v)
    {
        for (int i = x + 1; i <= n; i += i & -i)
        {
            a[i - 1] = a[i - 1] + v;
        }
    }

    T sum(int x)
    {
        T ans{};
        for (int i = x; i > 0; i -= i & -i)
        {
            ans = ans + a[i - 1];
        }
        return ans;
    }

    T rangeSum(int l, int r)
    {
        return sum(r) - sum(l);
    }

    int select(const T &k)
    {
        int x = 0;
        T cur{};
        for (int i = 1 << std::__lg(n); i; i /= 2)
        {
            if (x + i <= n && cur + a[x + i - 1] <= k)
            {
                x += i;
                cur = cur + a[x - 1];
            }
        }
        return x;
    }
};

void solve()
{
    int n;
    cin >> n;
    vector<int> prefix(n + 1);
    vector<int> arr(n + 1);
    vector<int> ans(n + 1);
    Fenwick<int> fk(n + 1);
    vector<int> vis(n + 1);
    prefix[1] = 1;
    i64 curans = 0;
    for (int i = 1; i <= n; i++)
    {
        cin >> arr[i];
    }
    vis[arr[1]]++;
    fk.add(arr[1], 1);
    for (int i = 2; i <= n; i++)
    {
        if (arr[prefix[i - 1]] < arr[i])
        {
            for (int j = i - 1; j >= prefix[i - 1]; j--)
            {
                if (--vis[arr[j]] == 0)
                {
                    fk.add(arr[j], -1);
                }
            }
            vis[arr[i]]++;
            fk.add(arr[i], 1);
            curans = ans[prefix[i - 1]] + 1;
            for (int j = prefix[i - 1] + 1; j <= i - 1; j++)
            {
                curans += fk.rangeSum(arr[j] + 1, n + 1);
                if (vis[arr[j]]++ == 0)
                {
                    fk.add(arr[j], 1);
                }
            }
            curans += fk.rangeSum(arr[prefix[i - 1]] + 1, n + 1);
            if (vis[arr[prefix[i - 1]]]++ == 0)
            {
                fk.add(arr[prefix[i - 1]], 1);
            }
            ans[i] = curans;
            prefix[i] = i;
        }
        else
        {
            curans += fk.rangeSum(arr[i] + 1, n + 1);
            ans[i] = curans;
            if (vis[arr[i]]++ == 0)
            {
                fk.add(arr[i], 1);
            }
            prefix[i] = prefix[i - 1];
        }
    }
    for (int i = 1; i <= n; i++)
    {
        cout << ans[i] << " \n"[i == n];
    }
}

signed main()
{
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);
    int t;
    cin >> t;
    while (t--)
    {
        solve();
    }
}
```
