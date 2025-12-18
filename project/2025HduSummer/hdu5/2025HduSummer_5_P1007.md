---
title: k-MEX
date: 2025-08-01
last_clear: 2025-09-08
tags:
  - tag/math/combinatorics
grasp: 2
difficulty:
optional_difficulty: Easy-Medium
source: https://acm.hdu.edu.cn/contest/problem?cid=1176&pid=1007
article:
---
# 题意

# Solution
![[Pasted image 20250801174152.png]]

$mex = i$ 的方案，是左边全选，$i$ 不选，剩下 $k-i$ 个数在剩下的 $n-i-1$ 个数里取出为  
$\binom{n-i-1}{k-i}$，因为 $0$ 对期望无贡献，从 $1$ 开始算，是  
$$
\frac{1}{\binom{n}{k}} \sum_{i=1}^k i \binom{n-i-1}{k-i}
$$
因为 $\binom{n}{k}$ 很大无法计算，我们想办法消了它
因为 $i$ 是系数，很烦人，用吸收率消了它
$$
\begin{align}
\sum_{i=1}^k i \binom{n-i-1}{k-i}
&= \sum_{i=1}^k i \binom{n-i-1}{n-k-1} \\
&= n \sum_{i=1}^k \binom{n-i-1}{n-k-1} - \sum_{i=1}^k (n-i) \binom{n-i-1}{n-k-1}
\end{align}
$$
反向利用吸收率
$$
= n \sum_{i=1}^k \binom{n-i-1}{n-k-1} - (n-k) \sum_{i=1}^k \binom{n-i}{n-k}
$$
接下来求
$$
\sum_{i=1}^k \binom{n-i-1}{n-k-1} = \binom{n-2}{n-k-1} + \cdots + \binom{n-k-1}{n-k-1},
$$
利用  $\binom{n}{m} = \binom{n-1}{m} + \binom{n-1}{m-1}$
在结尾加个 $\binom{n-k-1}{n-k} = 0$，就能往前一直合成为 $\binom{n-1}{n-k} = \binom{n-1}{k-1}$。  
同理，对于 $\sum_{i=1}^k \binom{n-i}{n-k}$，在结尾加 $\binom{n-k}{n-k+1} = 0$，就能合成 $\binom{n}{n-k+1} = \binom{n}{k-1}$。  
最后就是求
$$
n \binom{n-1}{k - 1} - (n-k) \binom{n}{k-1}
$$
$$
= n \binom{n}{k} \frac{k}{n} - (n-k) \binom{n}{k} \frac{k}{n-k+1}
= \binom{n}{k}(k - \frac{(n-k)k}{n-k+1})
$$
这样我们就把 $\binom{n}{k}$ 消了，答案是  
$$
k - \frac{(n-k)k}{n-k+1}
$$

或者，可以直接暴力展开
$$
n \binom{n-1}{k - 1} - (n-k) \binom{n}{k-1}
$$
$$
= n \times (n - 1)^{\underline{k-1}} / (k-1) ! - (n - k) \times n^{\underline{k-1}} / (k - 1)!
$$
$$
\binom{n}{k} = n^{\underline{k}} / k!
$$
相除得
$$
k - \frac{(n - k)k}{n - k + 1}
$$
#summary/combinatorics
# 组合数常用化简公式总结

## 基本恒等式

### 1. 吸收恒等式 (Absorption Identity)
$$
\begin{align}
\binom{n+1}{k+1} &= \frac{n+1}{k+1} \binom{n}{k} \\
\binom{n}{m} &= \frac{n - m + 1}{m} \binom{n}{m-1}
\end{align}
$$
**应用场景**：当组合数带有系数时，可以调整系数位置
### 2. 对称恒等式
$$
\binom{n}{k} = \binom{n}{n-k}
$$

**使用技巧**：计算时取 min(k, n-k) 可提高效率

## 求和恒等式

### 3. 平行求和 (Hockey-Stick Identity)
$$
\begin{align}
\sum_{i=r}^n \binom{i}{r} &= \binom{n+1}{r+1} \\
\sum_{i=0}^k \binom{n+i}{i} &= \binom{n+k+1}{k}
\end{align}
$$
**记忆方法**：求和结果的上指标+1

### 4. Vandermonde 卷积
$$
\sum_{k=0}^r \binom{m}{k}\binom{n}{r-k} = \binom{m+n}{r}
$$
**特例**：
$$
\sum_{k=0}^n \binom{n}{k}^2 = \binom{2n}{n}
$$
## 递推关系
### 5. Pascal 恒等式
$$
\binom{n}{k} = \binom{n-1}{k} + \binom{n-1}{k-1}
$$
**应用**：动态规划计算组合数的基础
## 二项式相关

### 6. 二项式定理推论
$$
\begin{align}
\sum_{k=0}^n \binom{n}{k} &= 2^n\\
\sum_{k=0}^n (-1)^k \binom{n}{k} &= 0 \quad (n \geq 1)
\end{align}
$$

### 7. 加权求和
$$
\begin{align}
\sum_{k=0}^n k \binom{n}{k} &= n2^{n-1} \\
\sum_{k=0}^n k^2 \binom{n}{k} &= n(n+1)2^{n-2}
\end{align}
$$
## 其他重要公式

### 8. 上指标反转
$$
\binom{-n}{k} = (-1)^k \binom{n+k-1}{k}
$$
### 9. 多项式系数
$$
\binom{n}{k_1,k_2,...,k_m} = \frac{n!}{k_1!k_2!...k_m!}
$$
(其中 $k_1 + k_2 + ... + k_m = n$)

## 使用策略

1. **系数处理**：优先考虑吸收恒等式
2. **求和化简**：
   - 单组合数求和 → Hockey-Stick
   - 组合数乘积求和 → Vandermonde
3. **递推分解**：使用 Pascal 恒等式
4. **对称优化**：计算时利用对称性