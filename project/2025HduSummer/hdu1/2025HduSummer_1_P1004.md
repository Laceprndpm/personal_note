---
title: 串串
date: 2025-07-18
last_clear: 2025-08-01
tags:
  - tag/string/SA
grasp: 1
difficulty: 2000
source: https://acm.hdu.edu.cn/contest/problem?cid=1172&pid=1004
article: https://oi-wiki.org/string/sa/
---
首先要知道后缀数组(SA)的几个性质
$$
lcp(sa[i], sa[j]) = range\_min\{height[i+1..j]\}
$$
那么，当我从前往后遍历 $sa$ 数组时，