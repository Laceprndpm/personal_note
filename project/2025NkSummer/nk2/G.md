---
title: Geometry Friend
date: 2025-07-21
last_clear: 2025-07-21
tags:
  - tag/计算几何
grasp: 3
difficulty: 1900
source: https://ac.nowcoder.com/acm/contest/108299/G
article:
---
#TODO/计算几何板子
# 题意
一个凸多边形懒洋洋地躺在欧几里得平面 $xOy$ 上。  
"拥有一个朋友是什么样的感觉呢？" 它自言自语道。  
一个小小的点 $P$ 出现在位置 $(x, y)$ 上，它引起了多边形的注意。它们成为了朋友，而多边形喜欢绕这个小点 $P$ 逆时针旋转（即以 $P$ 为旋转中心），因为这样它就可以覆盖一些以前从未覆盖过的区域。如果某个点在多边形的边缘或内部，则称这个点被多边形覆盖。  
但是 $P$ 感到担忧：有一天，多边形可能会厌倦绕着它旋转。它觉得，当多边形覆盖过的区域不再增大（也就是不再能覆盖新的点）时，多边形就会抛弃它，他们将不再是朋友。  
多边形的旋转角速度为每年 $1$ 弧度，设多边形开始旋转的时间为时间 $0$。请告诉 $P$ 什么时候多边形会不再是它的朋友，如果确实如它所想。
## 形式化
一个凸多边形和一个点，凸边形逆时针绕着点转，且每年转 1 弧度，求
凸边形覆盖区域达到最大值的最早时间。
# Solution
## std
凸多边形绕一个点覆盖的区域有两种可能——圆或圆环。
如果结果是一个圆环，我们考虑内径和外径。此时点一定在凸边形外侧。内径只需考虑图形中与点最近的一个点，这个点在凸多边形的情况
下是唯一的。所以此时一定需要绕一周，答案为 $2\pi$ 
如果结果是一个圆，则点在凸多边形的边缘或内部。此时只需考虑最外
侧圆何时完成覆盖。
最外侧的圆是由哪些点画出来的呢？离给定的点最远的一些点，这些点
一定是多边形的顶点。
因此取出所有最远的点后，按照极角排序，取相邻点之间的最大角度即
可。
时间复杂度为 O(n log n) 。也可以依据给的点是逆时针排列的，进一步
优化为 O(n) 。注意判断点是否在图形内最好使用整数判断。
## my
和std完全一致，缺乏计算几何板子，导致实现缓慢
```c++
constexpr int INF = 1e9;

struct Point {
    i64 x, y;

    friend Point operator-(Point& a, Point& b)
    {
        Point x{a.x - b.x, a.y - b.y};
        return x;
    }

    friend i64 operator*(const Point& a, const Point& b)
    {
        return a.x * b.y - a.y * b.x;
    }

    friend i64 dist2(Point& a, Point& b)
    {
        return (a.x - b.x) * (a.x - b.x) + (a.y - b.y) * (a.y - b.y);
    }
};

void solve()
{
    i64 n, x, y;
    cin >> n >> x >> y;

    vector<Point> point(n);
    for (int i = 0; i < n; i++) {
        cin >> point[i].x >> point[i].y;
    }
    Point p{x, y};
    bool  baohan  = 0;
    auto  contain = [&](Point a, Point b, Point c) {
        auto a_b = b - a, a_c = c - a, a_p = p - a, b_p = p - b, c_p = p - c;
        return abs(a_b * a_c) == abs(a_b * a_p) + abs(b_p * c_p) + abs(a_c * a_p);
    };

    for (int i = 1; i + 1 < n; i++) {
        if (contain(point[0], point[i], point[i + 1])) {
            baohan = 1;
            break;
        }
    }
    if (!baohan) {
        cout << "6.283185307179586\n";
        return;
    }
    vector<Point> bianjie;
    bianjie.pb(point[0]);
    for (int i = 1; i < n; i++) {

        if (dist2(p, bianjie[0]) < dist2(p, point[i])) {
            bianjie.clear();
            bianjie.pb(point[i]);
        } else if (dist2(p, bianjie[0]) == dist2(p, point[i])) {
            bianjie.pb(point[i]);
        }
    }
    vector<i64> dis;
    for (int i = 0; i < sz(bianjie); i++) {
        i64   tmpx = 2 * p.x - bianjie[i].x, tmpy = 2 * p.y - bianjie[i].y;
        Point mirror = {tmpx, tmpy};

        i64 cha = (bianjie[(i + 1) % sz(bianjie)] - bianjie[i]) * (mirror - bianjie[i]);
        dis.pb(dist2(bianjie[i], bianjie[(i + 1) % sz(bianjie)]));

        if (cha < 0) {
            i64    d2  = dis[i];
            i64    yao = dist2(p, bianjie[0]);
            double di = sqrtl(d2), yy = sqrtl(yao);
            double ans = 6.283185307179586 - asin(di / 2 / yy) * 2;
            printf("%.8lf\n", ans);
            return;
        }
    }
    int place1 = max_element(all(dis)) - bg(dis);
    int palce2 = (place1 + 1) % sz(bianjie);
    if (sz(bianjie) == 1) {
        cout << "6.283185307179586\n";
        return;
    }
    auto   one = bianjie[place1] - p, two = bianjie[palce2] - p;
    i64    d2  = dis[place1];
    i64    yao = dist2(p, bianjie[0]);
    double di = sqrtl(d2), yy = sqrtl(yao);
    double ans = asin(di / 2 / yy) * 2;
    printf("%.8lf\n", ans);
}
```
