---
title: Build a Computer
date: 2025-10-20
last_clear: 2025-10-20
tags:
  - QOJ/UCUP/3rd/Stage_14_Harbin
  - tag/数据结构/字典树
  - tag/数据结构/线段树
grasp: 3
value: 5
difficulty:
optional_difficulty: Medium
source: https://qoj.ac/contest/1817/problem/9519
article:
---
# Solution
感觉，其实就是01-trie套了个线段树
首先很容易联想到这样就可以构建出一个二进制段，即
`0bx0000` 到 `0bx1111` 
![[data/横向草稿(34).png]]
就和线段树一样了，每个节点负责了一个二进制段
然后注意到，所有的左区间其实对应线段树上的一条链，而移除前导零不影响这一性质，那么其实节点数就是 $\log(U) \times 2 + \log(U)$ ，算出为 $60$
![[横向草稿(35).png]]
最终代码跑出来上界为 $55$
```cpp showLineNumbers
constexpr int          N = 301;
vector<pair<int, int>> adj[N];
int                    range2[19]{-1, 2};
int                    curptr = 1;

int newNode();

void init(int x)
{
    if (x <= curptr) return;
    for (int i = curptr + 1; i <= x; i++) {
        range2[i] = newNode();
    }
    for (int i = x; i > curptr; i--) {
        adj[range2[i]].pb({range2[i - 1], 0});
        adj[range2[i]].pb({range2[i - 1], 1});
    }
    curptr = x;
}

int make_2range(int depth)
{
    init(depth);
    return range2[depth];
}

int tot = 0;

int trie[N][2];

int newNode()
{
    int x      = ++tot;
    trie[x][0] = trie[x][1] = 0;
    return x;
}

void add(int x, int t)
{
    int o = 1;
    for (int i = 29; i >= t; i--) {
        if (!(x >> i)) continue;
        int& p = trie[o][x >> i & 1];
        if (i == 0) {
            adj[o].pb({2, x & 1});
            return;
        }
        if (!p) {
            p = newNode();
            adj[o].pb({p, x >> i & 1});
        }
        o = p;
    }
    adj[o].pb({make_2range(t), 0});
    adj[o].pb({make_2range(t), 1});
}

void solve()
{
    newNode();
    newNode();
    // level = 19
    vector<pair<int, int>> op;
    auto                   dfs = [&](this auto&& self, int l, int r, int depth) -> void {
        if (l > r) return;
        int blockL = (l + (1 << depth) - 1) / (1 << depth) * (1 << depth);
        int blockR = blockL + (1 << depth) - 1;
        if (blockR <= r) {
            // tire.add(blockL, depth);
            op.pb({depth, blockL});
            self(l, blockL - 1, depth);
            self(blockR + 1, r, depth);
            return;
        }
        self(l, r, depth - 1);
    };
    int l, r;
    cin >> l >> r;
    dfs(l, r, 19);
    sort(all(op), greater<>());
    for (auto [depth, blockL] : op) {
        add(blockL, depth);
    }
    cout << tot << '\n';
    for (int i = 1; i <= tot; i++) {
        cout << adj[i].size() << ' ';
        for (auto [a, v] : adj[i]) {
            cout << a << ' ' << v << ' ';
        }
        cout << '\n';
    }
}
```
