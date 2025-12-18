---
title: 最甜的小情侣
date: 2025-09-12
last_clear: 2025-09-12
tags:
  - tag/math/分块
  - tag/math
grasp: 2
value: 2
difficulty:
optional_difficulty: Easy
source: https://acm.hdu.edu.cn/contest/problem?cid=1179&pid=1006
article: file:///home/patchouli/Voile/project/2025HduSummer/attachment/2025HduSummer/8/2025“钉耙编程”中国大学生算法设计暑期联赛（8）-标程+题解/2025“钉耙编程”中国大学生算法设计暑期联赛（8）-题解.pdf
---
# 题意
小 E 在仿照约瑟夫问题进行模拟游戏，游戏规则是这样的：
- 有 n 个人排成一排，每个人都有一个价值。游戏初始有一个参数 $1 \le w \le n$。
- n 个人的位置下标从 1 开始算，即 $1, 2, \cdots, n$。
- 每一轮，首先统计当前在场的所有人的价值和，计入 $sum$。
- 再查看当前剩余人数，如果 $< w$，游戏结束。
- 如果游戏没有结束，淘汰所有位置下标是 $w$ 的倍数的人，人的下标从 1 开始编号，所有淘汰结束后，未被淘汰的人按照从左到右重新编号。并重新开始新的一轮。

小 E 的目标是最大化最终得到的 $sum$，而 ta 唯一能决定的是初始每个人的价值。

已知这 n 个人的价值序列恰好是 $1, 2, \dots, n$ 的排列，位置可以任意安排，求能得到的最大 $sum$。
## Input
**本题有多组测试数据**。第一行一个正整数 `T`，表示数据组数，接下来输入每组测试数据。
对于每组测试数据：一行包含两个正整数 `n,w`，分别表示总人数和初始参数。
## Output
对于每组测试数据，输出一行一个正整数，表示能够获得的最大总价值。
## Solution
就推式子
![[2025HduSummer_8_P1006.png]]

```cpp showLineNumbers title="lace"
void solve()
{
    i128 n, w;
    cin >> n >> w;
    i128 pos = 1, round = 1;
    i128 ans = 0;
    while (n >= w) {
        i64 k = n / w;
        if (w * w > n) {
            i64  nextn    = k * w - 1;
            i128 blocknum = (n - nextn - 1) / k + 1;
            ans += i128(1) * ((blocknum - 1) * blocknum) / i128(2) * (k * pos + (k - 1) * k / 2);
            ans += i128(1) * ((blocknum - 1) * blocknum * (i128(2) * blocknum - 1)) / 6 * (k * k);

            ans += i128(1) * round * (pos + pos + blocknum * k - i128(1)) * (blocknum * k) / 2;
            round += blocknum;
            n -= blocknum * k;
            pos += blocknum * k;
        } else {
            ans += i128(1) * (pos + pos + k - 1) * k / 2 * round;
            pos += k;
            round++;
            n -= k;
        }
    }
    ans += (pos + n + pos - 1) * n / 2 * round;
    cout << ans << '\n';
}
```
