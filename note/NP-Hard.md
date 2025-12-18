# 图论中的经典 NP-Hard 问题

## 路径与游走相关
- Hamiltonian Path / Hamiltonian Cycle  
- 最长路径问题 (Longest Path)  
- 旅行商问题 (Traveling Salesman Problem, TSP)  
- 最小带权环覆盖 (Minimum Cycle Cover)  

## 顶点选择类
- 最大独立集 (Maximum Independent Set)  
- 最小顶点覆盖 (Minimum Vertex Cover)  
- 最大团 (Maximum Clique)  
- 支配集 (Dominating Set)  
- 最小反馈顶点集 (Minimum Feedback Vertex Set)  

## 边选择类
- 最小反馈边集 (Minimum Feedback Edge Set)  
- 边着色 (Edge Coloring, k ≥ 3)  
- 带约束的最大匹配 (Generalized Matching Problems)  

## 图划分 / 分割类
- 图着色 (Graph Coloring, k ≥ 3)  
- 团覆盖 (Clique Cover)  
- 图分割 (Graph Partitioning, e.g. Min Bisection, Sparsest Cut)  
- k-划分 (k-Partition)  

## 子图相关
- 子图同构 (Subgraph Isomorphism)  
- 最大公共子图 (Maximum Common Subgraph)  
- 最大诱导森林 (Maximum Induced Forest)  
- 最大诱导二部子图 (Maximum Induced Bipartite Subgraph)  

## 优化 / 网络类
- 最小 Steiner Tree  
- 网络设计问题 (Network Design Problems)  
- 带非线性费用的流问题 (Nonlinear Cost Flow Variants)  

## 总结规律
- 都是 **在图中选择点 / 边 / 子图，使目标函数极值化**。  
- 这些问题往往能归约自 Hamiltonian Path / Clique / Independent Set / Set Cover。  
- 优化版是 **NP-hard**，判定版通常是 **NP-complete**。  
- 在特殊图类（树、二部图、弦图等）可能有多项式算法。  
- 常见研究方向：近似算法、参数化算法 (FPT)、特殊图类。
