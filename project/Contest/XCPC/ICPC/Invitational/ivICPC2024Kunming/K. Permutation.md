---
title: Permutation
date: 2025-11-01
last_clear: 2025-11-01
tags:
  - QOJ/ICPC/Invitational/Kunming/2024
  - SUA
  - tag/交互
  - 红温
  - tag/细心
  - tag/概率
  - tag/概率/期望
  - tag/概率/随机
grasp: 2
value: 5
difficulty:
optional_difficulty: Medium
source: https://qoj.ac/contest/1802/problem/9432
article:
---
```cpp showLineNumbers title="jiangly"
#include <bits/stdc++.h>

using i64 = long long;
using u64 = unsigned long long;
using u32 = unsigned;

// std::vector<int> ans;

// int tot = 0;
int query(const std::vector<int> &p)
{
    // tot++;
    // int cnt = 0;
    // for (int i = 0; i < ans.size(); i++) {
    //     cnt += (p[i] == ans[i]);
    // }
    // return cnt;
    std::cout << 0;
    for (auto x : p) {
        std::cout << " " << x + 1;
    }
    std::cout << std::endl;
    int res;
    std::cin >> res;
    return res;
}

std::mt19937 rng(std::chrono::steady_clock::now().time_since_epoch().count());

int main()
{
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr);

    int n;
    std::cin >> n;

    // ans.resize(n);
    // std::iota(ans.begin(), ans.end(), 0);
    // std::shuffle(ans.begin(), ans.end(), rng);

    std::vector<int> p(n);
    auto             work = [&](auto &self, int l, int r, std::vector<int> a) -> void {
        if (r - l == 1) {
            p[l] = a[0];
            return;
        }
        int m = (l + r) / 2;

        std::shuffle(a.begin(), a.end(), rng);
        std::vector<int> v;

        std::vector<int> al, ar;
        for (auto x : a) {
            if (v.empty()) {
                v.push_back(x);
            } else {
                std::vector b(n, x);
                for (int i = l; i < m; i++) {
                    b[i] = v[0];
                }
                int res = query(b);
                if (res == 0) {
                    for (auto y : v) {
                        ar.push_back(y);
                    }
                    al.push_back(x);
                    v.clear();
                } else if (res == 1) {
                    v.push_back(x);
                } else {
                    for (auto y : v) {
                        al.push_back(y);
                    }
                    ar.push_back(x);
                    v.clear();
                }
            }
        }
        if (!v.empty()) {
            if (al.empty()) {
                al = v;
            } else if (ar.empty()) {
                ar = v;
            } else {
                std::vector b(n, al[0]);
                for (int i = m; i < r; i++) {
                    b[i] = v[0];
                }
                if (query(b) == 2) {
                    std::copy(v.begin(), v.end(), std::back_inserter(ar));
                } else {
                    std::copy(v.begin(), v.end(), std::back_inserter(al));
                }
            }
        }
        self(self, l, m, al);
        self(self, m, r, ar);
    };
    std::vector<int> a(n);
    std::iota(a.begin(), a.end(), 0);
    work(work, 0, n, a);

    // assert(p == ans);
    // std::cerr << "tot : " << tot << "\n";

    std::cout << 1;
    for (int i = 0; i < n; i++) {
        std::cout << " " << p[i] + 1;
    }
    std::cout << std::endl;

    return 0;
}
```
赛时以为 $\frac{2}{3} (n \cdot \log_{2}(n) - n) + n  / 2$ 能过 ，但是实际上是 $\frac{3}{4}$ 超次数
```cpp showLineNumbers title="myshit"
mt19937 rd(time(0));

void solve()
{
    int n;
    cin >> n;
    if (n == 1) {
        cout << 1 << ' ' << 1 << '\n';
        return;
    }
    vector<int> shoot(n + 1);
    iota(all(shoot), 0);
    shuffle(shoot.begin() + 1, shoot.end(), rd);
    int  askt = 0;
    auto ask  = [&](int l, int r, int a, int b) {
        if (askt >= 22222) {
            cout << "OOPS\n";
            exit(0);
        }
        cout << "0 ";
        int m = (l + r) / 2;
        for (int i = 1; i < m; i++) {
            cout << shoot[a] << ' ';
        }
        for (int i = m; i < n + 1; i++) {
            cout << shoot[b] << ' ';
        }
        cout << endl;
        askt++;
        int res;
        cin >> res;
        return res;
    };
    vector<int> ans(n + 1);
    auto        dfs = [&](auto&& self, int l, int r, vector<int> cur) -> void {
        // assert(r - l == cur.size());
        if (r - l != cur.size()) {
            cout << "r - l != cur.size()\n";
            exit(0);
        }
        shuffle(all(cur), rd);
        if (r == l) {
            cout << "L != R\n";
            exit(0);
        }

        if (r - l == 1) {
            ans[l] = cur[0];
            return;
        }
        if (r - l == 2) {
            int res = ask(l, r, cur[0], cur[1]);
            if (res == 2) {
                ans[l]     = cur[0];
                ans[l + 1] = cur[1];
            } else if (res == 0) {
                ans[l + 1] = cur[0];
                ans[l]     = cur[1];
            } else {
                cout << "114514\n";
                exit(0);
            }
            return;
        }
        int         m = (l + r) / 2;
        vector<int> lft, rgt;
        vector<int> unsure;
        int         i;
        for (i = 0; i < (r - l); i += 1) {
            if (lft.size() == (m - l)) {
                rgt.pb(cur[i]);
                continue;
            } else if (rgt.size() == r - m) {
                lft.pb(cur[i]);
                continue;
            }
            if (unsure.empty()) {
                unsure.pb(cur[i]);
            } else {
                int res = ask(l, r, unsure[0], cur[i]);
                if (res == 0) {
                    rgt.ins(rgt.end(), all(unsure));
                    lft.pb(cur[i]);
                    unsure.clear();
                } else if (res == 2) {
                    rgt.pb(cur[i]);
                    lft.ins(lft.end(), all(unsure));
                    unsure.clear();
                } else {
                    unsure.pb(cur[i]);
                }
            }
        }
        if (lft.size() < m - l) {
            lft.ins(lft.end(), all(unsure));
        } else {
            rgt.ins(rgt.end(), all(unsure));
        }
        self(self, l, m, lft);
        self(self, m, r, rgt);
    };
    vector<int> vec;
    auto        spask = [&](int a, int b) {
        askt++;
        cout << "0 ";
        for (int i = 1; i <= n - 1; i++) {
            cout << shoot[a] << ' ';
        }
        cout << shoot[b] << endl;
        int res;
        cin >> res;
        return res;
    };

    for (int i = 1; i <= n; i++) {
        vec.pb(i);
    }
    dfs(dfs, 1, n + 1, vec);
    for (int i = 1; i <= n; i++) {
        ans[i] = shoot[ans[i]];
    }
    cout << 1 << ' ';
    for (int i = 1; i <= n; i++) {
        cout << ans[i] << " ";
    }
    cout << endl;
}
```
