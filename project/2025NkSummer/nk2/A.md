---
title: Another Day of Sun
date: 2025-07-21
last_clear: 2025-07-21
tags:
  - tag/dp
grasp: 3
difficulty: 1400
source: https://ac.nowcoder.com/acm/contest/108299/A
article:
---
# 题意
但是作为一个睡眠爱好者，肖恩经常睡觉睡个半天，因此，当他醒来时，他甚至记不清今天是星期几。  
所以从某一天开始，他开始进行记录：每次他醒来的时候，他就会写下一个数字，表示此时外面是否有阳光。如果有，他会写一个 $1$；否则，他会写一个 $0$。在完成记录后，还没等太阳下山，或是太阳升起，他又再次入睡。假设每次肖恩醒来时，他看到的要么是阳光，要么是月光，但不会同时看到两者。  

这些记录下来的数字实际上形成了一个长度为 $n$ 的数组：  

$$
[a_1, a_2, \dots, a_n], \quad \forall 1 \le i \le n, \; 0 \le a_i \le 1
$$

其中 $a_i$ 表示肖恩写下的第 $i$ 个数字。  

然而，随着时间的推移，一些写下的数字变得模糊不清，你无法判断它是 $1$ 还是 $0$。如果有 $k$ 个数字无法识别，则可能有 $2^k$ 种不同的数组。  

对于每个可能的数组，你都可以依据这个数组计算肖恩看到阳光的最小天数。如果将可能的不同数组的最小天数结果相加，得到的结果如何呢？由于答案可能很大，请将结果关于 $998\,244\,353$ 取模后输出。
# Solution
```c++
void solve()
{
    int n;
    cin >> n;
    // 上一位为1的方案数， 上一位为0的方案数，如果是-1
    // 1 0 1 0 | 2
    // 1 0 1 1 | 2
    //
    // 1 1 1 1 | 1
    // 1 1 1 0 | 1
    vector<int> arr(n);
    for (int i = 0; i < n; i++) {
        cin >> arr[i];
    }
    Z   ans   = 0;
    i64 last0 = 1, last1 = 0;
    int cnt = 0;
    for (int i = 0; i < n; i++) {
        if (arr[i] == -1) {
            cnt++;
        }
    }
    Z base = power(Z(2), cnt);
    for (int i = 0; i < n; i++) {
        if (arr[i] == -1) {
            base /= 2;
            ans    = (ans + last1 * base);
            i64 n0 = (last0 + last1) % MOD;
            i64 n1 = (last0 + last1) % MOD;
            last0 = n0, last1 = n1;
        } else if (arr[i] == 1) {
            i64 n0, n1;
            n0    = 0;
            n1    = (last1 + last0) % MOD;
            last0 = n0, last1 = n1;
        } else {
            i64 n0, n1;
            ans   = (ans + last1 * base);
            n1    = 0;
            n0    = (last1 + last0) % MOD;
            last0 = n0, last1 = n1;
        }
    }
    ans = (ans + last1);
    cout << ans << '\n';
    cout.flush();
}
```