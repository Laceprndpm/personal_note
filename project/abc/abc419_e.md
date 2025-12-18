---
title:
date: 2025-08-16
last_clear: 2025-08-16
tags:
  - tag/thinking
grasp: 1
difficulty:
source:
article:
---
## E. **Subarray Sum Divisibility**

本题要求所有的长度为 $L$ 的区间和均为 $m$ 的倍数，根据此条件可以发现 $a_i \equiv a_{i + m} (\bmod m)$，原因如下。

列式子可得： $a_i + a_{i + 1} + \cdots + a_{i+m-1} \equiv 0, a_{i+1} + \cdots + a_{i+m-1} + a_{i+m} \equiv 0$

因此：$a_i + \sum\limits_{j = i +1}^{i +m -1}a_j \equiv \sum\limits_{j = i +1}^{i +m -1}a_j + a_{i + m} \equiv 0(\bmod  m)$。

消除相同项可得 $a_i \equiv a_{i + m} (\bmod m)$

所以，我们的第一步是将所有相距为m的数字视为一组，将他们变成除以m相同余数的数字。我们可以计算出每个位置$[0,L-1]$ 变成每一个余数 $[0, m- 1]$ 的最小花费。以上可以在 $O(nm)$ 时间内解决

第二步，我们只需要让前 $L$ 个数字的总和变为m的倍数即可。此时可以利用类似背包dp的思想。$dp_{i,j}$ 表示考虑前 $i$ 个数字，当前总和除以 $m$ 的余数为 $j$ 时的花费。注意此时的花费会同时计算后续的花费，即若 $n=10, L=3$ 则如果想让第 $0$ 个数字的余数为 $2$，则其花费为将第 $0,3,6,9$ 的位置的数字全部变为除以m余数为2的数字的花费。

```c++
#include<bits/stdc++.h>
using namespace std;
const int maxn = 2e6+5;
typedef long long ll;

int a[maxn];
int pre[maxn];
int cost[505][505];
int dp[505][505];

int main(){
    ios::sync_with_stdio(false);
    int n, m, k;
    cin >> n >> m >> k;
    for (int i = 0; i < n; i++) cin >> a[i];

    for (int i = 0; i < k; i++) {
        for (int j = 0; j < m; j++) {
            for (int t = i; t < n; t += k) {

                if (j - a[t] >= 0) cost[i][j] += j - a[t];
                else cost[i][j] += j - a[t] + m;
            }
        }
    }


    for (int i = 0; i <= k; i++) {
        for (int j = 0; j <= m; j++) dp[i][j] = 2e9;
    }
    dp[0][0] = 0;

    for (int i = 1; i <= k; i++) {
        // 之前的余数
        for (int j = 0; j < m; j++) {
            // 第i个数字产生的余数
            for (int t = 0; t < m; t++) {
                int mod = (j + t) % m; // 当前的余数
                int c = cost[i - 1][t] + dp[i - 1][j]; // 获得当前余数的花费
//                cout << i << " " << j << " "<< t << " " << mod << " " << c << endl;
                dp[i][mod] = min(dp[i][mod], c);
            }
        }
    }

    cout << dp[k][0] << endl;

}
```
# 反思
注意到了对隔 $m$ 位的数 $x_i + a_i \equiv x_{i + m} + a_{i + m}$ ，也注意到了只要确定前$L$ 项后面的也确定了，没有注意到等价于 $A_i \equiv A_{i + m}$ ，且dp的思考方向错了。~~因为忘记报名所以unrated所以没认真~~