---
title: 博弈
date: 2025-07-18
last_clear: 2025-07-18
tags:
  - tag/games
  - tag/games/antiNim
  - mystery
grasp: 2
difficulty: 1800
source: https://acm.hdu.edu.cn/contest/problem?cid=1172&pid=1001
article:
---
在做这题之前，我们需要知道 Anti_Nim：即在经典 Nim 游戏中，把获胜的条件改成 最后一个将石子取完的人
失败。我们有以下结论：
1. 当每堆石子的数量都为 1 时，异或和为 0 时先手获胜；
2. 当有至少一堆石子的个数大于 1 时，异或和不为 0 时先手获胜。
是第一个重要
```text
// 1为先手可以必胜,否则后手可以必胜，2为先手可以必败，否则后手可以必败
```
即1,2或-1,-2，一个是到这个地方的后手可以完全控制走向，一个是到这个地方的先手可以完全控制走向
1,-2或-1,2则只是颠倒
