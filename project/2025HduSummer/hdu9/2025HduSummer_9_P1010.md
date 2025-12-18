---
title: 阿斯蒂芬
date: 2025-09-06
last_clear: 2025-09-06
tags:
  - tag/图论/连通性相关/SCC
grasp: 3
difficulty:
optional_difficulty: Bronze
source: https://acm.hdu.edu.cn/contest/problem?cid=1180&pid=1010
article:
---
# 题意
原题略
## 形式化
给定一个有向图 $G=(V,E)$，其中顶点集合 $V=\{1,2,\dots,n\}$ 表示水晶。  
对于每个顶点 $i$，给定一个初始能量值 $a_i \in \mathbb{Z}_{\ge 0}$，以及一个布尔变量 $b_i \in \{0,1\}$ 表示该水晶的灵魂信标是否激活。  
此外，还给定邻接集合 $A_i=\{j\mid (i,j)\in E\}$，即能量从 $i$ 传递到的所有顶点。
系统按照如下离散步骤无限执行，直到终止：
1. **激活阶段**：  
   定义集合  
   $$
   J=\{k\in V \mid a_k>0\}
   $$
   若 $J=\varnothing$，过程终止。
2. **能量流动阶段**：  
   对于每个 $k\in J$，执行：
   - $a_k \leftarrow a_k - 1$。  
   - 若 $b_k=1$，则总能量池 $T \leftarrow T+1$。  
   - 对每个 $j\in A_k$，令 $a_j \leftarrow a_j + 1$。  
   如果过程无限进行，则称系统发生了 **能量失控**，此时 $T$ 会无限增长。  
   为了避免能量失控，可以对某些顶点 $i$ 设置 $b_i \leftarrow 0$，即关闭对应的灵魂信标。  
   **任务：**
   在保持 $T$ 有限的前提下，求最少需要关闭多少个灵魂信标。  
## Input
第一行：整数 $A$（$1 \leq A \leq 100$），表示测试数据组数。
对于每组测试数据：
- 第一行：整数 $n$（$1 \leq n \leq 5 \times 10^5$）
- 第二行：$n$ 个自然数 $a_1, a_2, \dots, a_n$（初始能量值）
- 第三行：$n$ 个自然数 $b_1, b_2, \dots, b_n$（每个 $b_i$ 表示集合 $A_i$ 的大小）
- 接下来 $n$ 行：第 $i$ 行包含 $b_i$ 个整数，表示集合 $A_i$（即与水晶 $i$ 相连的水晶编号）
### 数据规模约束
- 对于每组测试数据：$n \leq 5 \times 10^5$，且所有集合 $A_j$ 的大小总和 $\sum b_j \leq 3 \times 10^6$
- 所有测试数据累计：$n$ 的总和 $\sum n \leq 2.5 \times 10^6$，所有集合 $A_j$ 的大小总和 $\sum b_j \leq 1.5 \times 10^7$
- 初始能量值 $a_i \leq 2$（对于所有 $i$）
### Output
$A$ 行，每行一个非负整数，表示需要将灵魂信标设置为“沉寂”状态的最少个数。
# Solution
## std
将每个魔法水晶视作一个点，则诸个 $A_j$ 按照 $j \to k \in A_j$ 诱导出一张有向图 $G$。设 $G = (V, E)$。下文称包含至少两个点的 SCC 为 **真 SCC**。

对于 $G$ 中的任意一个真 $\text{SCC} \; S \subseteq V$，一旦任一 $x \in S$ 在某时刻 $a_x > 0$，则整个 SCC 的过程不能停止，并且还会为其出边不断贡献能量。我们称这种真 SCC 为**可激活的**。于是问题转化为：找所有这样的点 $x$，使得对所有可激活的真 $\text{SCC} \; S$，不存在 $S$ 到 $x$ 的路径。我们记这样的 $x$ 构成集合 $U \subseteq V$；所有 $V \setminus U$ 中的点皆要被设置为“沉寂”。

容易发现，一个真 $\text{SCC} \; S$ 是可激活的，如果它本身包含 $a_x > 0$ 的点 $x$，或者有任何一个点 $y$ 满足 $a_y > 0$ 且存在 $y$ 到 $S$ 的路径。对 $G$ 缩点得到 $G'$ 后，对每个点 $x \in G'$，按照拓扑排序的顺序维持是否有在前的点有能量能流到 $x$；若能，且 $x$ 在原图是一个真 SCC，则它就是可激活的，所有以它出发能到达的点皆要被设置为“沉寂”。
## my
想想看哪些情况下会无限输出，归根结底只有两种情况
```mermaid
graph TD
    Node_1((1)) --> Node_2((2)) --> Node_3((3))
    Node_2((2)) --> Node_1((1))
    

    %% 使用 style 直接定义内联样式
    classDef A fill:#f88,stroke:#333,stroke-width:2px;
    classDef B fill:#0000ff,stroke:#333,stroke-width:2px;
    
    %% 新增：定义控制尺寸和字体的类
    classDef bigNode width:60px, height:60px, font-size:48px;
    
    class Node_1,Node_2 A;
    class Node_3 B;
    
    %% 为所有节点应用大尺寸样式
    class Node_1,Node_2,Node_3 bigNode;
```
红色代表本身具有回环
蓝色表示上游节点带有回环

同时，值得注意的是，如果上游节点且回环本身不具有能量，则根本不需要禁用。
由此，scc后缩点即可。
```cpp showLineNumbers title="P1010.cpp"
void solve()
{
    int n;
    cin >> n;
    vector<int> arr(n);
    vector<int> brr(n);
    for (int i = 0; i < n; i++) {
        cin >> arr[i];
    }
    for (int i = 0; i < n; i++) {
        cin >> brr[i];
    }
    vector<vector<int>> adj(n);
    SCC                 scc(n);
    for (int i = 0; i < n; i++) {
        int bi = brr[i];
        for (int j = 0; j < bi; j++) {
            int v;
            cin >> v;
            v--;
            adj[i].pb(v);
            scc.addEdge(i, v);
        }
    }
    scc.work();
    vector<int>            vis(n);
    int                    cnt = scc.cnt;
    vector<int>            val(cnt);
    vector<int>            contain_num(cnt);
    vector<pair<int, int>> edge;
    function<void(int)>    dfs1 = [&](int u) -> void {
        if (vis[u]) return;
        vis[u] = 1;
        contain_num[scc.bel[u]]++;
        val[scc.bel[u]] += arr[u];
        for (int v : adj[u]) {
            if (scc.bel[v] != scc.bel[u]) {
                // new_graph[scc.bel[u]].ins(scc.bel[v]);
                edge.pb({scc.bel[u], scc.bel[v]});
            }
            dfs1(v);
        }
    };
    for (int i = 0; i < n; i++) {
        dfs1(i);
    }
    sor(edge);
    edge.erase(unique(all(edge)), edge.end());
    vector<vector<int>> new_graph(cnt);
    for (auto [u, v] : edge) {
        new_graph[v].pb(u);
    }
    i64                    ans = 0;
    vector<array<bool, 2>> dp(cnt);  // 0表示有值，1表示有环该删
    vector<int>            visted(cnt);
    function<void(int)>    dfs2 = [&](int u) -> void {
        if (visted[u]) return;
        visted[u] = 1;
        if (val[u]) dp[u][0] = 1;
        for (int v : new_graph[u]) {
            dfs2(v);
            if (dp[v][0]) {
                dp[u][0] = 1;
            }
            if (dp[v][1]) {
                dp[u][1] = 1;
            }
        }
        if (dp[u][0] && contain_num[u] >= 2) {
            dp[u][1] = 1;
        }
        if (dp[u][1]) {
            ans += contain_num[u];
        }
        return;
    };
    for (int i = 0; i < cnt; i++) {
        if (!visted[i]) {
            dfs2(i);
        }
    }
    cout << ans << '\n';
}
```
