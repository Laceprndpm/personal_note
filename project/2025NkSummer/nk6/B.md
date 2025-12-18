---
title: Base Conversion Master
date: 2025-09-02
last_clear: 2025-09-02
tags:
  - tag/模拟
grasp: 3
difficulty: 1800
source: https://ac.nowcoder.com/acm/contest/108303/B
article:
---
# 题意
时间限制：1 秒  
内存限制：256 MB  
小 T 得到了 $n$ 个数组 $A_1, \dots, A_n$，第 $i$ 个数组 $A_i$ 的长度为 $l_i$。  
另外，小 T 给出了两个整数 $y$ 和 $M$，希望你可以确定所有在 $[2, M]$ 内满足以下要求的正整数 $s$：

- 按顺序枚举 $i \in [1, n]$：  
  - 将数组 $A_i$ 看作 $s$ 进制下的数字。如果数组 $A_i$ 中存在一个不小于 $s$ 的数字，则认为 $s$ 不满足要求。  
  - 对 $A_i$ 进行进制转换，得到结果 $s'$, 并赋值给 $s$。

初始值 $s$ 满足要求，当且仅当完成上述 $n$ 次进制转换后有 $s = y$。

具体来说，给定一个数组 $a_1, \dots, a_k$ 以及初始进制 $s$，则将数组作为 $s$ 进制下的数字转换后得到：

$$
s' = \sum_{i=1}^{k} a_i s^{k-i} = a_1 s^{k-1} + a_2 s^{k-2} + \cdots + a_k
$$

小 T 的研究发现，满足条件的正整数 $s$要么不存在，要么恰好为某个区间 $[l, r]$ 内的所有正整数。  

你的任务是：  
- 向小 T 汇报对应的 $l, r$，或者告诉他不存在满足条件的正整数 $s$。
## Input
- 第一行包含一个整数 $T$ $(1 \le T \le 10^5)$，表示测试数据的组数。  

对于每组测试数据：

- 第一行包含三个整数 $n, y, M$ $(1 \le n \le 2 \times 10^5, 1 \le y \le 10^9, 2 \le M \le 10^9)$。  
- 接下来的 $n$ 行，第 $i$ 行首先包含一个整数 $l_i$ $(1 \le l_i \le 2 \times 10^5)$，表示数组 $A_i$ 的长度。紧接着包含 $l_i$ 个整数 $A_{i,j}$ $(0 \le A_{i,j} \le 10^9)$，表示数组 $A_i$ 的内容。保证数组的第一个元素 $A_{i,1} \ne 0$。  

数据保证：一次输入中所有数组的长度和不超过 $2 \times 10^5$。
## Output
- 对于每组测试数据：  
  - 如果不存在满足要求的正整数 $s$，输出一行 `-1 -1`。  
  - 否则，假设所有满足要求的正整数形成区间 $[l, r]$，则输出一行两个整数 $l$ 和 $r$。
## 形式化
给定 $n$ 个数组 $A_i$，目标正整数 $y$ 和进制上界 $M$。  
对于一个进制 $s \in [2, M]$，判断通过如下流程能否得到 $y$：
- 按顺序枚举每个数组 $A_i$，将其看作 $s$ 进制下的数字，并将得到的结果 $s'$ 赋值给 $s$。  
- 如果中途出现数组中存在数字 $\ge s$ 或无法完成进制转换，则认为无法得到 $y$。  
可以证明满足条件的 $s$ 必然形成一个区间。  
任务：给出这个区间，或者判断不存在满足条件的 $s$。  
- $1 \le n \le 2 \cdot 10^5$  
- $1 \le y \le 10^9$  
- $2 \le M \le 10^9$
# Solution
## std
可以发现，对于两个进制 $s_1, s_2$, 如果有 $s_1 < s_2$, 那么 $s_1$ 在操作后得到的
数字也不大于 $s_2$ 操作后得到的数字。这启发我们进行二分。

对于二分的中间值 $s$, 需要计算出从 $s$ 开始进行操作得到的数字。如果中间发现无法进制转换,
直接返回 $-1$。

二分找到第一个结果不小于 $y$ 的位置 $l$, 再二分找到第一个结果大于 $y$ 的位置 $r$,
利用 $l, r$ 的大小关系即可得到答案。时间复杂度
$O\big(\sum l_i \log M\big)$, 其中 $l_i$ 是数组 $A_i$ 的长度。
```cpp showLineNumbers title="lace"
void solve()
{
    u32 n, y, M;
    cin >> n >> y >> M;
    vector<u32>         len_vec(n);
    vector<vector<u32>> vec(n);
    for (u32 i = 0; i < n; i++) {
        cin >> len_vec[i];
        vector<u32> tmp(len_vec[i]);
        for (u32 j = 0; j < len_vec[i]; j++) {
            cin >> tmp[j];
        }
        vec[i] = tmp;
    }
    for (u32 i = 0; i < n; i++) {
        reverse(all(vec[i]));
    }
    auto check = [&](u64 s) -> int {
        for (u32 i = 0; i < n; i++) {
            u64 tmp   = 0;
            u64 cur_s = 1;
            if (s == UINT64_MAX) {
                if (len_vec[i] == 1) {
                    s = vec[i][0];
                }
                continue;
            }
            for (u32 j = 0; j < len_vec[i]; j++) {
                if (vec[i][j] >= s) {
                    return -1;
                }
            }
            for (u32 j = 0; j < len_vec[i]; j++) {
                tmp += cur_s * vec[i][j];
                if ((cur_s * s > (1e9 + 5) && j != len_vec[i] - 1) || tmp > 1e9) {
                    tmp = UINT64_MAX;
                    break;
                }
                cur_s *= s;
            }
            s = tmp;
        }
        if (s < y) {
            return -1;
        } else if (s == y) {
            return 0;
        } else {
            return 1;
        }
    };
    if (check(2) == 1 || check(M) == -1) {
        cout << "-1 -1\n";
        return;
    }
    u32 lval = 2, rval = 1e9 + 5;
    u32 lower;
    while (lval <= rval) {
        u32 mid = (rval - lval) / 2 + lval;
        if (check(mid) >= 0) {
            rval  = mid - 1;
            lower = mid;
        } else {
            lval = mid + 1;
        }
    }
    u32 higher;
    lval = 2, rval = 1e9 + 5;
    while (lval <= rval) {
        u32 mid = (rval - lval) / 2 + lval;
        if (check(mid) <= 0) {
            lval   = mid + 1;
            higher = mid;
        } else {
            rval = mid - 1;
        }
    }
    higher = min(higher, M);
    if (higher < lower) {
        cout << "-1 -1\n";
        return;
    }
    cout << lower << ' ' << higher << '\n';
}
```