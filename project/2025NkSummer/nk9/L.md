---
title: Ping Pong
date: 2025-09-05
last_clear: 2025-09-05
tags:
  - tag/打表
grasp: 3
difficulty:
optional_difficulty: Easy
source: https://ac.nowcoder.com/acm/contest/108306/L
article:
---
# 题意
有 $n$ 个同学轮流在一张乒乓球桌上打球。第 $i$ 个人的能力值是 $a_i$，所有能力值两两不同。
一开始，场上有且仅有一个同学，编号为 $1$。队列 $Q=\{2,3,\dots,n\}$ 表示当前排队等待的同学，按从前到后的顺序排列。
在接下来的每一轮中，队列最前面的同学会出列并与场上的同学比赛。比赛过程中，能力值较高的同学获胜。输家会加入队列的末尾，而赢家会继续留在场上。
为了防止强力同学垄断比赛，设有反垄断规则：如果某人已经连续参加了 $n-1$ 场比赛，那么接下来的比赛无论结果如何都会被判定为该人的失败；他们会加入队列的末尾，而他们的对手会继续留在场上。
他们将进行总共 $k$ 轮比赛。求每个人在这 $k$ 轮中分别打了多少场比赛。
约束条件：
- $\sum n \le 10^6$；
- $1 \le k \le 10^9$。
## Input
输入包含多个测试用例。第一行包含一个正整数 $T$ $(1 \le T \le 10000)$ 表示测试用例的数量。每个测试用例描述如下：

- 第一行包含两个整数 $n, k$ $(3 \le n \le 2\times 10^5,\ 1 \le k \le 10^9)$。
- 第二行包含 $n$ 个整数 $a_1, a_2, \dots, a_n$ $(1 \le a_i \le 10^9)$。

保证所有 $a_i$ 两两不同，并且 $T$ 个测试用例中 $n$ 的总和不超过 $2\times 10^5$。

## Output
对于每个测试用例，输出一行，包含 `n` 个整数。
第 `i` 个整数表示 输入顺序中第 i 个同学已经参与过的比赛总数。
## 形式化
有 $n$ 个同学排队打乒乓球，每个人有一个能力值 $a_i$，且能力值互不相同。比赛时，能力值高的一方会获胜。
每一轮比赛由队首的两人进行，胜者留在场上，败者则回到队尾。  
存在一个特殊规则：如果某名选手连续打了 $n-1$ 场比赛，下一场视作他输，结束后下场排到队尾。
现在比赛进行了 $k$ 轮，求每个同学在这 $k$ 轮中分别打了多少场比赛。
约束条件：
- $\sum n \le 10^6$；  
- $1 \le k \le 10^9$。 
有 n 个同学排队打乒乓球，每个人有一个能力值 ai，且能力值互不相
同。比赛时，能力值高的一方会获胜。
# Solution
## my
通过打表可以发现
```text
who = 
[
  [5, 4],
  [5, 3],
  [5, 2],
  [5, 1],
  [5, 4],
  [4, 3],
  [4, 2],
  [4, 1],
  [4, 5],
  [5, 3],
  [5, 2],
  [5, 1],
  [5, 4],
  [4, 3],
  [4, 2],
  [4, 1],
  [4, 5],
  [5, 3],
  [5, 2],
  [5, 1],
  [5, 4],
  [4, 3],
  [4, 2],
  [4, 1],
  [4, 5],
  [5, 3],
  [5, 2],
  [5, 1],
  [5, 4],
  [4, 3],
  [4, 2],
  [4, 1],
  [4, 5],
  [5, 3],
  [5, 2],
  [5, 1],
  [5, 4],
  [4, 3],
  [4, 2],
  [4, 1],
  [4, 5],
  [5, 3],
  [5, 2],
  [5, 1],
  [5, 4]
]
```
存在循环节，因为不难发现，在最大的打败这 `n-1` 个人后，他会立即失败，此时这 `n-1` 个人内战后，剩下的一定是第二强的，而第二强的会在撞到最强的后立即失败，而最强的在打败 `n-1` 个时一定是最后一个打败第二强的。那么后面的流程一定是第二强的打败剩下 `n-2` 个人，然后被最强的打败。
```cpp showLineNumbers title="lace"
void solve()
{
    int n, k;
    cin >> n >> k;

    vector<int> arr(n);
    for (int i = 0; i < n; i++) {
        cin >> arr[i];
    }
    int        mx_val = max_element(all(arr)) - arr.begin();
    deque<int> que(n);
    iota(all(que), 0);
    vector<array<int, 2>> who(9 * n);
    int                   playerA = que.front(), playerB = -1;
    que.pop_front();
    int wintime = 0;
    for (int i = 0; i < 9 * n; i++) {
        playerB = que.front();
        que.pop_front();
        who[i][0] = playerA;
        who[i][1] = playerB;
        if (arr[playerA] > arr[playerB] && wintime < n - 1) {
            wintime++;
            que.push_back(playerB);
        } else {
            wintime = 1;
            que.push_back(playerA);
            playerA = playerB;
        }
    }
    int startp = max(mx_val - 1, 0);
    while (who[startp][0] == mx_val || who[startp][1] == mx_val) {
        startp++;
    }
    while (who[startp][0] != mx_val && who[startp][1] != mx_val) {
        startp++;
    }
    vector<int> times(n);
    int         bglen = min(startp, k);
    k -= bglen;
    for (int i = 0; i < bglen; i++) {
        for (int j = 0; j < 2; j++) {
            times[who[i][j]]++;
        }
    }
    int epochs = k / (2 * n - 2);
    for (int i = 0; i < 2 * n - 2; i++) {
        int j = i + startp;
        times[who[j][0]] += epochs;
        times[who[j][1]] += epochs;
    }
    k -= epochs * (2 * n - 2);
    for (int i = 0; i < k; i++) {
        int j = i + startp;
        times[who[j][0]]++;
        times[who[j][1]]++;
    }
    for (int i = 0; i < n; i++) {
        cout << times[i] << ' ';
    }
    cout << '\n';
}
```
## std
如果不考虑特殊规则，那么当最强的选手上场之后，他会一直连胜下去，其他人会轮流和他对战。

考虑特殊规则后，最强的选手在连续对战 $n-1$ 轮后需要下场，接下来轮到其他人继续比赛，直到最强选手再次轮到上场。这个过程中会形成一个循环节，循环节的长度将会恰好是 $2n-2$。

模拟前 $4n$ 轮比赛，找到这个循环节即可。复杂度 $O(n)$。
