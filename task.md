每日任务
```dataview
table title, last_clear, grasp, source
from ""
where last_clear and last_clear <= date(today) - dur(14 days) and grasp <= 2
sort grasp asc
```
```dataview
table title, last_clear, grasp, source
from ""
where  grasp = 1
sort grasp asc
```
- [ ] [# P2617 Dynamic Rankings][https://www.luogu.com.cn/problem/P2617]树状数组套主席树
- [ ]  P5284