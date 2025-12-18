---
title: Good Partitions
date: 2025-10-21
last_clear: 2025-10-21
tags:
  - QOJ/ICPC/Regionals/Chengdu
  - tag/数据结构/线段树
  - tag/math/exgcd
  - tag/thinking/reverse
grasp: 3
value: 5
difficulty:
optional_difficulty: Medium
source: https://qoj.ac/contest/1821/problem/9543
article: https://codeforces.com/group/iF1v4Ff8lz/contest/631372/attachments/download/28066/ICPC%E6%88%90%E9%83%BD.pdf
---
相当于维护当前所有割点gcd
赛时用 $N\sqrt{(N)}$ 反向维护非割点草过去的，但是对线段树可以有更深理解，而不是紧紧浮于表面、只靠经验。
![[Pasted image 20251021005229.png]]
![[Pasted image 20251021005504.png]]
![[Pasted image 20251021005255.png]]
![[Pasted image 20251021005520.png]]
![[Pasted image 20251021005541.png]]
#TODO 