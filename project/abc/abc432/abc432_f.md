---
title: F - Candy Redistribution
date: 2025-11-15
last_clear: 2025-11-15
tags:
  - tag/bitmasks
  - tag/dp/状压dp
grasp: 0
value: 5
difficulty:
optional_difficulty: Easy-Medium
source: https://atcoder.jp/contests/abc432/tasks/abc432_f
article:
---
这里如果直接用 `s` 表示处理过和没处理过的会导致状态不能唯一表示，但是注意到，每次转移时一定会将一个非 `0` 的数变成 `0` ，可以枚举上一个状态来将状态锁死，感觉是可以进一步拓展的小巧思。