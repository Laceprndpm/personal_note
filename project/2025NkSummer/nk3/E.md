---
title: Equal
date: 2025-07-22
last_clear: 2025-09-01
tags:
  - tag/hash
  - tag/math/数论
grasp: 3
difficulty: 1900
source: https://ac.nowcoder.com/acm/contest/108300/E
article:
---
# 题意
Yuki 给了你一个长度为 $n$ 的正整数序列 $a_1, \dots, a_n$。你可以做以下两种操作任意次：
- 选择正整数 $i, j, d$ 满足 $1 \le i < j \le n$ 且 $d \mid a_i, d \mid a_j$，然后将    $$
  a_i \leftarrow \frac{a_i}{d}, \quad a_j \leftarrow \frac{a_j}{d}
  $$
- 选择正整数 $i, j, d$ 满足 $1 \le i < j \le n$，然后将  
  $$
  a_i \leftarrow a_i \cdot d, \quad a_j \leftarrow a_j \cdot d
  $$
判断通过若干次操作后，能否使  
$$
a_1 = a_2 = \dots = a_n
$$

# Solution
## my
和std一样，简单的说就是所有数中的每种质因子的总数的奇偶性不变，因为每次要么同时对 $a, b$ 乘上 $d$ ，一定加偶数，要么同时除以 $d$ ，一定减偶数。
代码不建议参考，std中有hash优化版本
```c++
void solve()
{
    int n;
    cin >> n;
    vector<int> arr(n);
    for (int i = 0; i < n; i++) {
        cin >> arr[i];
    }
    if (n % 2) {
        cout << "YES\n";
        return;
    }
    if (n == 2) {
        if (arr[0] != arr[1]) {
            cout << "NO\n";
            return;
        } else {
            cout << "YES\n";
            return;
        }
    }
    bool ok = 1;
    for (int p : prime.primes) {
        int cnt = 0;
        for (int i = 0; i < n; i++) {
            while (arr[i] % p == 0) {
                arr[i] /= p;
                cnt++;
            }
        }
        if (cnt % 2) {
            ok = 0;
        }
    }
    map<int, int> remain;
    for (int i : arr) {
        if (i != 1) {
            remain[i]++;
        }
    }
    for (auto [_, i] : remain) {
        if (i % 2) {
            ok = 0;
        }
    }
    if (ok) {
        cout << "YES\n";
    } else {
        cout << "NO\n";
    }
}
```
## std
让我们分别考虑每个质因子 $p^k$，因为它们总是独立的。取对数 $\ln$ 使我们能够将乘法和除法视为加法和减法。可以观察到：

$$
\left(\sum k\right) \bmod 2
$$

是一个不变量，从而可以发现一个必要条件：要么 $2 \nmid n$，要么 $2 \mid \sum k$ 且 $n > 2$，要么 $n = 2$ 且 $k_1 = k_2$。

这个必要条件也是充分的，我们可以做一个构造性证明：

- 当 $(\sum k) \bmod 2 = 0$ 时：

  - 当 $n = 2$ 时，如果 $k_1 \ne k_2$，同时加减操作均无法使 $k_1 = k_2$。  
  - 当 $n > 2$ 时，考虑反复选择两个位置 $k_i, k_j > 0$，并同时将它们减去 1，直至无法选择。此时最多会有一个位置满足 $k_i > 0$（如果没有，则已经达到目标）。  
    接着选择三个两两不同的位置 $i, i_1, i_2$ 并依次执行以下操作：  
    $$
    k_{i_1}, k_{i_2} \text{加 } 1; \quad k_i, k_{i_1} \text{减 } 1; \quad k_i, k_{i_2} \text{减 } 1
    $$
    由于 $(\sum k) \bmod 2$ 是不变的，$a_i$ 在此过程中始终为偶数，最终一定能减少到 0，从而使所有 $k_i$ 相等。
- 当 $2 \nmid n$ 时：
  - 当 $n = 1$ 时，所有数字已经相等。  
  - 当 $n > 1$ 时，如果 $(\sum k) \bmod 2 = 0$，可以执行与上述相同的过程。否则，我们仍然可以使用相同方法让数组 $k$ 包含恰好 $n-1$ 个 0 和 1 个 1，通过将 $n-1$ 个 0 配对，将它们全部增加到 1，从而使所有 $k_i$ 相等。

设 $V = \max(a_i)$。朴素实现的瓶颈在于质因数分解，时间复杂度为 $O(n\sqrt{V})$。然而，我们只需要检查所有质数出现的次数是否为偶数，因此可以使用 XOR Hashing。具体做法是：  

1. 为每个质数设置一个随机哈希值；  
2. 使用线性筛初始化所有合数的哈希值。  

这样整体时间复杂度为 $O(n + V)$。
### mycode:
```c++
mt19937                            rng(time(0));
std::uniform_int_distribution<int> dist(1, INT32_MAX);

struct Prime {
    vector<int> minp, primes;
    vector<int> xor_hash;

    Prime(int n)
    {
        sieve(n);
    }

    void sieve(int n)
    {
        minp.assign(n + 1, 0);
        xor_hash.assign(n + 1, 0);
        primes.clear();

        for (int i = 2; i <= n; i++) {
            if (minp[i] == 0) {
                minp[i]     = i;
                xor_hash[i] = dist(rng);
                primes.push_back(i);
            }

            for (auto p : primes) {
                if (i * p > n) {
                    break;
                }
                minp[i * p]     = p;
                xor_hash[i * p] = xor_hash[i] ^ xor_hash[p];
                if (p == minp[i]) {
                    break;
                }
            }
        }
    }
} p1(5e6 + 5), p2(5e6 + 5);

void solve()
{
    int n;
    cin >> n;
    vector<int> arr(n);
    for (int& i : arr) {
        cin >> i;
    }
    if (n % 2 == 1) {
        cout << "Yes\n";
        return;
    }
    if (n == 2 && arr[0] != arr[1]) {
        cout << "No\n";
        return;
    }
    int hash1 = 0, hash2 = 0;
    FOR(i, n)
    {
        hash1 ^= p1.xor_hash[arr[i]];
        hash2 ^= p2.xor_hash[arr[i]];
    }
    if (hash1 == 0 && hash2 == 0) {
        cout << "Yes\n";
    } else {
        cout << "No\n";
    }
}
```
# 总结
当只关心奇偶时，可以用异或hash来处理，原理是对于每一位，异或相当于$\pmod2$ 下加法。
