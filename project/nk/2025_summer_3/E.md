---
title: Equal
date: 2025-07-22
last_clear: 2025-07-22
tags:
  - tag/hash
grasp: 3
difficulty: 1900
source: https://ac.nowcoder.com/acm/contest/108300/E
article:
---
可以用素数筛+hash
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