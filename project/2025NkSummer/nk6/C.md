---
title: Stack
date: 2025-09-02
last_clear: 2025-09-02
tags:
grasp:
difficulty:
source: https://ac.nowcoder.com/acm/contest/108303/C
article:
---
# 题意
对于一个排列 `P` ，定义 `f(P)` 如下：
```py showLineNumbers title="f"
function f(P):
	stack = []
	
	for element in P:
		while stack is not empty and stack.top() > element:
			stack.pop()
		stack.push(element)
		
	return the size of stack
```
给定一个整数 n 。找到长度为 n 的所有排列中 $\left(f\left(P′\right)\right)^3$ 的和，模 998244353。
# Solution
考虑 $f(P)$ 的值
假如$P$是
对于全排列
$$
\sum_{\forall P}{(f(P))^3} = n! \cdot E[(f(P))^3]
$$
其中$E(\ (f(P))^3\ )$ 是期望
记
$$
f(P) = \sum_{i = 1}^{i \le n} x_i
$$
其中
$$
x_i = 
\begin{cases}
1 & \text{if } i \text{ 在单调栈中} \\
0 & \text{otherwise}
\end{cases}
$$
那么
$$
E[x^3] = E\left[ \left(\sum_{i = 1}^{i \le n}x_i\right)^3 \right]
$$
展开后就可以得到
$$
\begin{align}
E[x^3] 
&= E[\sum_{i = 1}^n{x_i}^3 + 3\sum_{i = 1}^n{x_i}^2x_j + \sum_{\begin{aligned}i \neq j\\j\neq k\\ i \neq k\end{aligned}}x^ix^jx^k] \\
&= \sum_{i = 1}^{n}E[x_i] + 6 \sum_{1\le i < j \le n}E[x_ix_j] + 6 \sum_{i < j < k}E[x_ix_jx_k]
\end{align}
$$

$$
\begin{align}
E[x_i]
&= 1 \cdot P(x_i = 1) + 0 \cdot P(x_i = 0)\\
&= P(x_i = 1) \\
&= \frac{1}{i}
\end{align}
$$
$$
\begin{align}
E[x_ix_j]
&= 1 \cdot P(x_i = 1 \text{ and } x_j = 1) \\
&\text{这里先确定j的位置可以获得 1/j 的概率} \\
&\text{再确定i的位置在此基础上，获得 1/i 的概率} \\
&= \frac{1}{ij}
\end{align}
$$

$$
\begin{align}
E[x_ix_jx_k] 
&= P(x_i = 1 \text{ and } x_j = 1 \text{ and } x_k = 1) \\
&= \frac{1}{ijk}
\end{align}
$$

$$
E[x^3] = \sum_{i=1}^n\frac{1}{i} + 6\sum_{i < j}{\frac{1}{ij}} + 6\sum_{i < j < k}\frac{1}{ijk}
$$
```cpp showLineNumbers title="lace"
void solve()
{
    auto invv = []() {
        array<Z, MAXN + 1> tmp{};
        tmp[1] = 1;
        for (u32 i = 2; i <= MAXN; i++) {
            tmp[i] = -tmp[P % i] * (u32(P) / i);
        }
        return tmp;
    }();
    vector<Z> dp1(MAXN + 1), dp2(MAXN + 1), dp3(MAXN + 1);
    for (u32 i = 1; i <= MAXN; i++) {
        assert(i < sz(dp1));
        dp1[i] = dp1[i - 1] + invv[i];
    }
    for (u32 i = 1; i <= MAXN; i++) {
        dp2[i] = dp2[i - 1] + invv[i] * dp1[i - 1];
    }
    for (u32 i = 1; i <= MAXN; i++) {
        dp3[i] = dp3[i - 1] + invv[i] * dp2[i - 1];
    }
    int t;
    cin >> t;
    while (t--) {
        int x;
        cin >> x;
        cout << comb.fac(x) * (dp1[x] + 6 * dp2[x] + 6 * dp3[x]) << '\n';
    }
}
```
