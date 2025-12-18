---
title: 括号匹配
date: 2025-09-06
last_clear: 2025-09-06
tags:
  - tag/字符串/Manacher
grasp: 3
difficulty:
optional_difficulty: Iron
source: https://acm.hdu.edu.cn/contest/problem?cid=1180&pid=1006
article:
---
# 题意
本题中字符串下标从 $1$ 开始。
对于字符串 $S$，我们称 $S$ 的**非空子串**为其中连续一段字符构成的字符串。第 $l$ 个字符到第 $r$ 个字符（$l \le r$）构成的子串记作 $S[l\ldots r]$。
称非空字符串 $S$ 是**回文串**，当且仅当把所有字符的顺序翻转过来后 $S$ 保持不变。例如 `()()` 不是回文串，而 `())(` 是回文串。
称非空字符串 $S$ 是**括号串**，当且仅当 $S$ 中只有 `(` 和 `)` 且两种字符一样多，并且对于任意的 $i$，$S[1\ldots i]$ 中 `(` 的数量不少于 `)` 的数量。例如 `()` 是括号串，而 `())(()` 不是括号串。
给出一个字符串 $S$，求 $S$ 有多少个子串 $T$ 同时满足：
- $T$ 是回文串；
- $T$ 中存在一个子串是括号串。
两个子串的下标范围不同，则视为不同的子串。
## Input
输入一行一个字符串 $S$（$1 \le |S| \le 10^4$，$S$ 中仅含小写字母和小括号），含义同题目描述。
## Output
输出一个自然数，表示答案。
## Examples
input
```txt
()(a)a()(
```
output
```txt
4
```
## Hint
- `()( ` 是回文子串，其中 `()` 是括号串。由于该子串出现了两次，所以记作两个。
- 另外两个符合条件的子串分别是 `)(a)a()` 和 `()(a)a()(`。
注意：子串 `(a)a(` 虽然是回文子串，但其中不包含括号串，因此不计入答案。
# Solution
## my
马拉车后，对于每个中心点，找到包含括号串的范围和马拉车的回文子串的范围，差即是这个中心点可以产生的所有答案。
通过赛后总结和阅读他人代码，建议马拉车都使用奇偶分类讨论。
赛时误解：以为`(a)` 也是回文子串，导致时间浪费。
```cpp showLineNumbers title="P1006.cpp"
void solve()
{
    string s;
    cin >> s;
    std::string t = "#";
    for (auto c : s) {
        t += c;
        t += '#';
    }
    // begin manacher
    int              n = t.size();
    std::vector<int> r(n);
    for (int i = 0, j = 0; i < n; i++) {
        if (2 * j - i >= 0 && j + r[j] > i) {
            r[i] = std::min(r[2 * j - i], j + r[j] - i);
        }
        while (i - r[i] >= 0 && i + r[i] < n && t[i - r[i]] == t[i + r[i]]) {
            r[i] += 1;
        }
        if (i + r[i] > j + r[j]) {
            j = i;
        }
    }
    // end manacher

    vector<int> cloest_left(n, INF);
    int         last = -INF;
    for (int i = 0; i < n; i++) {
        cloest_left[i] = min(cloest_left[i], i - last);
        if (i + 2 < n && t[i] == '(' && t[i + 2] == ')') {
            last = i;
        }
    }
    vector<int> cloest_right(n, INF);
    last = INF;
    for (int i = n - 1; i >= 0; i--) {
        cloest_right[i] = min(cloest_right[i], last - i);
        if (i - 2 > 0 && t[i] == ')' && t[i - 2] == '(') {
            last = i;
        }
    }
    vector<int> atleast(n);
    for (int i = 0; i < n; i++) {
        atleast[i] = min(cloest_left[i], cloest_right[i]);
    }
    i64 ans = 0;
    for (int i = 0; i < n; i++) {
        int big_r   = r[i];
        int small_r = atleast[i];
        ans += max(big_r - small_r + 1, 0) / 2;
    }
    cout << ans << '\n';
}
```
