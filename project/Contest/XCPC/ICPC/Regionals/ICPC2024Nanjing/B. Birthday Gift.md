---
title: Birthday Gift
date: 2025-10-31
last_clear: 2025-10-31
tags:
  - QOJ/ICPC/Regionals/Nanjing/2024
  - SUA
  - tag/贪心
  - tag/脑电波
  - tag/thinking
  - tag/thinking/奇偶性
  - tag/thinking/attention
grasp: 2
value: 5
difficulty:
optional_difficulty: Medium
source: https://qoj.ac/contest/1828/problem/9565
article:
---
注意到奇偶性不变后，需要将所有奇数位上 `1` 变成 `0` ，然后就变成每次消去不同的数字，那么此时，最终的字符串一定是 `00000221111` 这样，那么2只需要变成少的那个数字就可以正确。
不能理解为什么只要做映射就可以这么快解决。。。
推测：
1. 消去相同的，本身比消去不同的更加复杂
2. 奇偶用在 `01` 字符串，刚刚好都是 2 的等价类
3. 在新的串中，原来能消去的相邻数（奇偶不同且相同）会变成 “不同且奇偶不同”，它只是让所有“可消去的对”都表现为“相同字符”，方便统计。简单地说，原本是两个二分图--按01分图和按奇偶性连边，被压成了一个只需要考虑01的二分图，方便统计。

他人题解：
>奇数位的两个 0 一定不能互相消去，相邻（也就是奇偶性不同的）两个 0 一定可以消去。我们记录互消之后剩下的 0 和 1，贪心地将 2 变成 0 和 1 即可。
>官方题解先将奇数位 01 互换，本质是一样的。

尝试证明不同奇偶的相同数一定可以消去
如果不能消去，则存在中间有偶数个数都不能消去，那么必须是 `0101` 串，那么必有一端可能继续消去，证毕
```cpp showLineNumbers
void solve()
{
    string s;
    cin >> s;
    int cnt0 = 0, cnt1 = 0;
    int cnt2 = 0;
    for (int i = 0; i < s.size(); i++) {
        if (s[i] == '2') {
            cnt2++;
            continue;
        }
        if (((s[i] - i) % 2 + 2) % 2) {
            cnt1++;
        } else {
            cnt0++;
        }
    }
    if (cnt0 > cnt1) swap(cnt0, cnt1);
    int add = min(cnt1 - cnt0, cnt2);
    cnt0 += add;
    cnt2 -= add;
    cnt0 += cnt2 / 2;
    cnt2 -= cnt2 / 2;
    cnt1 += cnt2;
    cout << s.size() - 2 * min(cnt0, cnt1) << '\n';
}
```