---
title: Ghost in the Parentheses
date: 2025-07-24
last_clear: 2025-07-24
tags:
  - tag/etc/括号序列
  - tag/thinking/差分与前缀和
grasp: 2
difficulty: 1900
source: https://ac.nowcoder.com/acm/contest/108301/G
article:
---
# 题意
给一个合法括号序列 s，每个字符以 1/2的概率变成问号字符。问能唯
一还原合法括号序列的概率。
- $|s| \le 10^6$ 
# Solution
## std
将 `'('` 替换为 $+1$，`)` 替换为 $-1$。  
子序列满足条件当且仅当：
1. 子序列为若干个 $+1$ 与若干个 $-1$ 的拼接；
2. 若该子序列既有 $+1$，又有 $-1$，令 $i$ 为子序列中最后一个 $+1$ 在原序列中的位置，$j$ 为子序列中第一个 $-1$ 在原序列中的位置，则交换原序列中第 $i$ 号位置与第 $j$ 号位置，序列的前缀和不再始终非负。  

条件 2 等价于  
$$
\min_{i \le k < j} \text{prefix\_sum}_k \le 1.
$$

利用这些条件，我们只需要枚举 $i$ 的位置，找到最小的 $j$ 满足上述条件，即可在 $O(1)$ 时间内计数。  
总的时间复杂度是 $O(|s|)$ 或 $O(|s|\log |s|)$。
## my
假如已经选了 $n$ 个 `(` 在前， $m$ 个 `)` 在后。假如想构造新的括号序列，明显的交换最后一个 `(` 和第一个 `)` 最优的。
交换操作相当于什么？
对于i到j区间，使得前缀和在\[i, j)上-2（因为不仅仅失去了一个+1,还多了一个-1）,而前缀和始终不能 $<0$ 。
```c++
void solve()
{
    // (( (xx()()) )) -> 0.75 * ()(()())
    // 1 -1 1 -1
    //
    // ((()(()())))
    // for left in range n
    // C(prefix, left)
    //
    // 1 1 1 -1 1 1 -1 1 -1 -1 -1 -1
    // 1 2 3  2 3 4  3 4  3  2  1  0
    // 如果st不混淆，s+t是否一定不混淆？
    // (?+?) x
    // 如果st混淆，s+t是否一定混淆？
    // ✔
    // 如果s混
    // 定义：一个合法的括号序列是独立的
    //
    // （每个(的前缀和为0||每个)的后缀和为0）&&所有(在)前，充分必要
    //
    // 1 2 1 0
    // ( ( ) )
    // ( ? ? )
    //
    // 交换操作相当于什么？
    // 对于i_j区间，使得前缀和在[i, j)上-2（因为不仅仅失去了一个+1,还多了一个-1）,而前缀和始终不能<0
    //
    //
    //
    string s;
    cin >> s;
    int         n = len(s);
    vector<int> prefix(n + 1);
    for (int i = 0; i < n; i++) {
        prefix[i + 1] = prefix[i];
        if (s[i] == '(') {
            prefix[i + 1]++;
        } else {
            prefix[i + 1]--;
        }
    }
    vector<int> cnt_right(n + 1);
    vector<int> cnt_left(n + 1);
    for (int i = 0; i < n; i++) {
        cnt_right[i + 1] = cnt_right[i];
        cnt_left[i + 1]  = cnt_left[i];
        if (s[i] == ')')
            cnt_right[i + 1]++;
        else
            cnt_left[i + 1]++;
    }
    SegmentTree<Info> seg(prefix);

    auto f = [](Info x) -> bool {
        return x.x <= 1;
    };
    Z ans = 0;
    for (int i = 0; i < n; i++) {
        if (s[i] == ')') continue;
        // prefix[i + 1, fi_r] <= 1
        // r > fi_r - 1
        // r >= fi_r
        int fi_r    = seg.findFirst(i + 1, n, f) - 1 + 1;
        Z   cur_ans = power(Z(2), cnt_left[i]);
        cur_ans *= (power(Z(2), cnt_right[n] - cnt_right[fi_r]) - 1);
        ans += cur_ans;
    }
    ans += power(Z(2), n / 2 + 1);
    ans -= 1;
    ans /= power(Z(2), n);
    cout << ans << '\n';
}
```
