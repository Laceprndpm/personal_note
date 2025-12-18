---
title: 计算几何
date: 2025-09-06
last_clear: 2025-09-06
tags:
  - tag/计算几何
grasp: 3
difficulty:
optional_difficulty: Bronze
source: https://acm.hdu.edu.cn/contest/problem?cid=1180&pid=1005
article: file:///home/patchouli/Voile/project/2025HduSummer/attachment/2025HduSummer/9/2025“钉耙编程”中国大学生算法设计暑期联赛（9）-标程+题解/2025“钉耙编程”中国大学生算法设计暑期联赛（9）-题解.pdf
---
# 题意
给定一个网格图和一个闭合不自交的折线，问应该如何使用尽可能少的钉子和一条弹性圈来模拟出这条折线。钉子的半径忽略不计。
## Input
- 第一行包含一个正整数 $T$ $(1 \le T \le 100)$，表示数据组数。  

对于每组数据：

- 第一行包含一个正整数 $n$ $(3 \le n \le 5\times 10^5,\ \sum n \le 5\times 10^5)$，表示折线的折点个数。  
- 接下来 $n$ 行，每行包含两个整数 $x_i, y_i$ $(-2\times 10^9 \le x_i, y_i \le 2\times 10^9)$，表示第 $i$ 个点的坐标（点编号从 $1$ 到 $n$）。  

说明：这里的折线为闭合折线  
$$(x_1,y_1)-(x_2,y_2)-\dots-(x_n,y_n)-(x_1,y_1).$$  
注意：不保证任意三个相邻点不共线。
## Output
对于每组数据，输出如下内容：

- 首先输出一个正整数 $k$，表示至少需要使用的钉子数；  
- 接着输出 $k$ 行，每行包含三个项：兩个整数 $x,y$ 和一个字符串（`YES` 或 `NO`），表示一枚钉子的位置 $(x,y)$ 以及该钉子是否被折线所“包裹”。  
- 这 $k$ 个钉子需按 $x$ 从小到大排序；当 $x$ 相同时按 $y$ 从小到大排序。  
- “被折线包裹”定义为：该点位于由折线形成的多边形的内部（输出 `YES`）；否则输出 `NO`。

每组数据以上述格式独立输出。
图一：
![[2025HduSummer_9_P1006_0.webp]]
# Solution
## std
大家首先需要意识到，在一条折线的每个拐点处，必须在突出侧内部放置一个钉子，不然的话弹性绳就会收缩使得最终形状不符合预期。
但是由于题目问的是每个点在折线包成的多边形内部还是外部，所以我们还需要判断这些线段的左边是多边形内部还是右边是多边形内部。
判断方法可以是去判断折线从开始到最后是顺时针转了一圈（每条线段右边是多边形内部）还是逆时针转了一圈（每条线段左边是多边形内部）
我们可以看到，样例一折线是顺时针转了一圈，每条线段右边是多边形内部；样例二折线是逆时针转了一圈，每条线段左边是多边形内部。
到这里，题目就做完了。
```cpp showLineNumbers title="P1005_std.cpp"
#include <bits/stdc++.h>
#define ll          long long
#define pb          push_back
#define fi          first
#define se          second
#define db          long double
#define mp          make_pair
#define fo(i, l, r) for (ll i = l; i <= r; i++)
using namespace std;
const ll M   = 1e6 + 7;
const db Eps = 1e-20;
ll       n, x[M], y[M], cnt, X[M], Y[M], In[M], id[M];
db       rd;

db ang(ll x1, ll y1, ll x2, ll y2) { return atan2(x1 * y2 - x2 * y1, x1 * x2 + y1 * y2); }

bool cmp(ll x, ll y) { return X[x] < X[y] || (X[x] == X[y] && Y[x] < Y[y]); }

int main()
{
    // freopen("geometry.in","r",stdin);
    // freopen("geometry.out","w",stdout);
    int T;
    cin >> T;
    while (T--) {
        scanf("%lld", &n), cnt = 0, rd = 0;
        fo(i, 1, n) scanf("%lld%lld", &x[i], &y[i]);
        x[n + 1] = x[1], y[n + 1] = y[1];
        x[n + 2] = x[2], y[n + 2] = y[2];
        fo(i, 1, n) { rd += ang(x[i + 1] - x[i], y[i + 1] - y[i], x[i + 2] - x[i + 1], y[i + 2] - y[i + 1]); }
        // rd 负：顺时针转；正：逆时针转
        fo(i, 1, n)
        {
            if (ang(x[i + 1] - x[i], y[i + 1] - y[i], x[i + 2] - x[i + 1], y[i + 2] - y[i + 1]) > Eps)
                cnt++, X[cnt] = x[i + 1], Y[cnt] = y[i + 1], In[cnt] = 1;
            if (ang(x[i + 1] - x[i], y[i + 1] - y[i], x[i + 2] - x[i + 1], y[i + 2] - y[i + 1]) < -Eps)
                cnt++, X[cnt] = x[i + 1], Y[cnt] = y[i + 1], In[cnt] = 0;
        }
        if (rd < 0)
            for (int i = 1; i <= cnt; i++) In[i] = 1 - In[i];
        for (int i = 1; i <= cnt; i++) id[i] = i;
        sort(id + 1, id + 1 + cnt, cmp);
        printf("%lld\n", cnt);
        for (int i = 1; i <= cnt; i++) {
            printf("%lld %lld ", X[id[i]], Y[id[i]]);
            if (In[id[i]] == 1)
                puts("YES");
            else
                puts("NO");
        }
    }
    return 0;
}
```
