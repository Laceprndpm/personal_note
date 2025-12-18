---
title: "10010"
date: 2025-09-08
last_clear: 2025-09-08
tags:
  - tag/hash
  - tag/二分搜索
  - tag/数据结构/树状数组
  - tag/模拟
grasp: 0
value: 4
difficulty: 1900
optional_difficulty: Easy-Medium
source: https://acm.hdu.edu.cn/contest/problem?cid=1173&pid=1011
article:
---
# 题目描述

对于一个二进制数 $x(>0)$，设 $2^y = \text{lowbit}(x)$，设 $z = \lfloor x/2^{y+1} \rfloor$。

定义函数：$f(0)=0$，当 $x>0$ 时

$$
f(x) = 
\begin{cases} 
y & z=0 \\
f(z)+2 & z \neq 0 \land \text{lowbit}(z)=\text{lowbit}(x)\times 2 \\
y & z \neq 0 \land \text{lowbit}(z)\neq \text{lowbit}(x)\times 2
\end{cases}
$$

给出一个长度为 $n$ 的 01 序列 $a$。有 $m$ 次操作，每次操作都形如以下的两种：

- `1 l r` ：查询序列 $a$ 的区间 $[l,r]$ 组成的二进制数 $x$ 的 $f(x)$ 值。
- `2 x` ：令 $a_x$ 反转（0 变成 1, 1 变成 0）。

> $\text{lowbit}(i)$ 表示：在二进制表示下最低位的 1 及其后面所有的 0 构成的数值。  
> $\land$ 表示逻辑与。

---

## Input

每个测试点中包含多组测试数据。输入的第一行包含一个正整数 $T(1 \leq T \leq 110)$，表示数据组数。对于每组测试数据：

第一行两个正整数 $n,m(1 \leq n \leq 5 \times 10^5, 1 \leq m \leq 5 \times 10^5)$，分别表示字符串长度和操作次数。

第二行一个长度为 $n$ 的字符串 $a$，表示初始的 01 序列。

接下来 $m$ 行，每行若干个整数，第一数 opt 表示操作类型：

- 若 opt=1，则后面两个整数 $l,r(1 \leq l \leq r \leq n)$，表示查询的区间。
- 若 opt=2，则后面一个整数 $x(1 \leq x \leq n)$，表示修改的位置。

保证所有测试数据中 $n$ 之和不超过 $6.2 \times 10^5$，$m$ 之和不超过 $7.1 \times 10^5$。

---

## Output

对于每组测试数据：对于每一个 opt=1 的操作，输出一行一个数表示答案。
# Solution
其实分享一下题意，就是对于一个二进制数，每次去除最低的一个 `1` ，并且如果下一个 `1` 的位权刚刚好是上一个的位权 `+1` 就可以继续维持下去。
举个例子，对于1000100010010，首先反转得到
`0100100010001`
然后就是 1 2，到3时，下一个也是3，所以无法继续下去，答案就是
3 + 2 + 2 = 7
## hash+二分(倍增)
在给每个位置分配位权后
假设想匹配 $001000100001...$ (已反转)
则需要构造
```python showLineNumbers
init = 0
begin = 3, end = 5
for i in range(begin, end + 1):
	init = 0
	hash += base ** i
```
记hash1为来自构造
$$
\begin{align}
\text{hash1} 
&= base^{(3)} + base^{(3 + 4)} + base^{(3 + 4 + 5)} \\
&= (base^{(1)} + base^{(1 + 2)} + base^{(1 + 2 + 3) }+ base^{(1 + 2 + 3 + 4)} + base^{(1 + 2 + 3 + 4 + 5)} \\
	&- base^{(1)} + base^{(1 + 2)}) \times \mathrm{inv}(base^{(1 + 2)})
\end{align}
$$
而hash2直接来自于hash数组，只需要 $\times \mathrm{inv}(\text{pos})$ 来实现 $\text{shift}$ 
在下面代码实现中，因为base并非取模，不存在 $\mathrm{inv}$ ，采用了双方同乘的方法消去逆元。
### 难点分析
ChatGPT:
哈希构造公式
这里用了双层前缀和：
$\text{position\_power[i]} = base^i$ ——单点的权值；
$\text{power\_idx[i]} = base^{(1+2+...+i)}$ ——即位权序列的“指数和”；
$\text{prefix\_pi[i]} = \sum \text{power\_idx[j]}$ ——整个序列的前缀和，用于快速计算一个连续的“链式反应哈希”。
check 函数的比较，就是把“理想链式序列”的哈希（来自 prefix_pi）和“真实区间序列”的哈希（来自 Fenwick）对齐，看是否一致。
这一块数学推导很绕，大多数人会卡在“为什么是三层前缀”的地方。
>我的理解就是，在知道了他是一个等差数列累加的情况下，如何从前缀和快速还原
>我们知道 **链式反应里的位权 hash** 本质是一个「等差数列的幂次累加」。难点就在于：如何把这个式子写成前缀和的形式，避免每次 $O(\sqrt{n})$ 去构造。

```cpp showLineNumbers title="std_hash"
#include <bits/stdc++.h>
#define ll  long long
#define pb  push_back
#define u64 unsigned long long
using namespace std;
const int N       = 5.2e5 + 5;
const u64 MODBASE = 131;
u64       position_power[N], power_idx[N], prefix_pi[N];
ll        idx[N];
int       n, m, a[N];
string    s;

u64 qpow(u64 x, ll y)
{
    u64 z = 1;
    while (y) {
        if (y & 1) z *= x;
        x *= x, y >>= 1;
    }
    return z;
}

namespace Fen {
int num_1[N], fr;
u64 hash_arr[N];

void init()
{
    memset(num_1, 0, (n + 1) * 4);
    memset(hash_arr, 0, (n + 1) * 8);
}

void add(int x, int y, u64 val)
{
    for (; x <= n; x += x & -x) num_1[x] += y, hash_arr[x] += val;
}

u64 sum_hash(int r)
{
    u64 res = 0;
    for (; r; r -= r & -r) res += hash_arr[r];
    return res;
}

u64 sumT(int l, int r) { return sum_hash(r) - sum_hash(l - 1); }

int sum_num_1(int r)
{
    int res = 0;
    for (; r; r -= r & -r) res += num_1[r];
    return res;
}

int sumA(int l, int r) { return sum_num_1(r) - sum_num_1(l - 1); }

bool check(int L, int suma, u64 sumt)
{
    if (!suma) return true;
    int p2 = fr + suma - 1;
    u64 bb = prefix_pi[p2] - prefix_pi[fr - 1];  // fr 为第一个1前0的数量，p2为最后一个1前0的数量
    // init = 0
    // for (i, fr, p2 + 1):
    //     init += i
    //     hash1 += qpow(base, init)
    // 
    u64 tt = sumt;
    tt *= power_idx[fr];
    bb *= position_power[L + fr];
    return bb == tt;
}

int twopoint(int L)
{
    int x    = 0;
    int suma = -sum_num_1(L - 1);
    u64 sumt = -sum_hash(L - 1);
    for (int i = 31 - __builtin_clz(n); ~i; --i) {
        x ^= 1 << i;
        if (x > n || (x >= L && !check(L, suma + num_1[x], sumt + hash_arr[x])))
            x ^= 1 << i;
        else
            suma += num_1[x], sumt += hash_arr[x];
    }
    return x;
}

int two0(int L)
{
    int x    = 0;
    int suma = -sum_num_1(L - 1);
    for (int i = 31 - __builtin_clz(n); ~i; --i) {
        x ^= 1 << i;
        if (x > n || (x >= L - 1 && suma + num_1[x]))
            x ^= 1 << i;
        else
            suma += num_1[x];
    }
    return x - L + 1;
}
}  // namespace Fen

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    position_power[0] = 1;
    for (int i = 1; i < N; ++i) position_power[i] = position_power[i - 1] * MODBASE;
    idx[0] = 1;
    for (int i = 1; i < N; ++i) idx[i] = idx[i - 1] + i + 1;
    for (int i = 0; i < N; ++i) power_idx[i] = qpow(MODBASE, idx[i]);
    prefix_pi[0] = power_idx[0];
    for (int i = 1; i < N; ++i) prefix_pi[i] = prefix_pi[i - 1] + power_idx[i];
    int T;
    cin >> T;
    while (T--) {
        cin >> n >> m;
        cin >> s;
        for (int i = 1; i <= n; ++i) a[i] = s[n - i] - '0';
        Fen::init();
        for (int i = 1; i <= n; ++i)
            if (a[i]) Fen::add(i, 1, position_power[i]);
        while (m--) {
            int opt;
            cin >> opt;
            if (opt == 1) {
                int l, r;
                cin >> r >> l;
                l = n - l + 1, r = n - r + 1;
                int fr  = 0;
                fr      = Fen::two0(l);
                Fen::fr = fr;
                if (l + fr > r) {
                    cout << "0\n";
                    continue;
                }
                int R = Fen::twopoint(l);
                if (R > r) R = r;
                int res = Fen::sumA(l, R);
                cout << res * 3 + fr - 3 << "\n";
            } else {
                int x;
                cin >> x;
                x = n - x + 1;
                a[x] ^= 1;
                if (a[x])
                    Fen::add(x, 1, position_power[x]);
                else
                    Fen::add(x, -1, -position_power[x]);
            }
        }
    }
    return 0;
}
```
