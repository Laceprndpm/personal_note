---
title: Italian Cuisine
date: 2025-10-31
last_clear: 2025-10-31
tags:
  - QOJ/ICPC/Invitational/Kunming/2024
  - SUA
  - tag/计算几何/旋转卡壳
grasp: 4
value: 3
difficulty:
optional_difficulty: Easy-Medium
source: https://qoj.ac/contest/1802/problem/9434
article:
---
计算几何板子题
```cpp showLineNumbers
using T = long long;  // 全局数据类型

constexpr T eps = 0;
constexpr T INF = numeric_limits<T>::max();

struct Point {
    T x, y;

    bool operator==(const Point &a) const { return (abs(x - a.x) <= eps && abs(y - a.y) <= eps); }

    bool operator<(const Point &a) const
    {
        if (abs(x - a.x) <= eps) return y < a.y - eps;
        return x < a.x - eps;
    }

    bool operator>(const Point &a) const { return !(*this < a || *this == a); }

    Point operator+(const Point &a) const { return {x + a.x, y + a.y}; }

    Point operator-(const Point &a) const { return {x - a.x, y - a.y}; }

    Point operator-() const { return {-x, -y}; }

    Point operator*(const T k) const { return {k * x, k * y}; }

    Point operator/(const T k) const { return {x / k, y / k}; }

    T operator*(const Point &a) const { return x * a.x + y * a.y; }  // 点积

    T operator^(const Point &a) const { return x * a.y - y * a.x; }  // 叉积，注意优先级

    int toleft(const Point &a) const
    {
        const auto t = (*this) ^ a;
        return (t > eps) - (t < -eps);
    }  // to-left 测试

    T len2() const { return (*this) * (*this); }  // 向量长度的平方

    T dis2(const Point &a) const { return (a - (*this)).len2(); }  // 两点距离的平方
};

// 多边形
struct Polygon {
    vector<Point> p;  // 以逆时针顺序存储

    size_t nxt(const size_t i) const { return i == p.size() - 1 ? 0 : i + 1; }

    size_t pre(const size_t i) const { return i == 0 ? p.size() - 1 : i - 1; }

    // 多边形面积的两倍
    // 可用于判断点的存储顺序是顺时针或逆时针
    T area() const
    {
        T sum = 0;
        for (size_t i = 0; i < p.size(); i++) sum += p[i] ^ p[nxt(i)];
        return sum;
    }
};

i64 xc, yc, r;

// 直线
struct Line {
    Point p, v;  // p 为直线上一点，v 为方向向量

    Line(Point a, Point b)
    {
        p = a;
        v = b - a;
    }

    int toleft(const Point &a) const { return v.toleft(a - p); }

    bool dis(const Point &a) const
    {
        return i128(abs(v ^ (a - p))) * abs(v ^ (a - p)) / (v * v) >= (r * r);
    }  // 点到直线距离
};

struct Convex : Polygon {

    // 旋转卡壳
    // 例：凸多边形的直径的平方
    T rotcaliper() const
    {

        const auto  area = [](const Point &u, const Point &v, const Point &w) { return (w - u) ^ (w - v); };
        const auto &p    = this->p;
        if (p.size() == 1) return 0;
        if (p.size() == 2) return p[0].dis2(p[1]);
        T     res = 0;
        T     cur = 0;
        Point center;

        center.x = xc, center.y = yc;
        for (size_t i = 0, j = 1; i < p.size(); i++) {
            const auto nxti = this->nxt(i);
            while (Line(p[i], p[this->nxt(j)]).dis(center) && Line(p[i], p[this->nxt(j)]).toleft(center) > 0) {
                cur += area(p[i], p[j], p[this->nxt(j)]);
                j = this->nxt(j);
            }
            res = max(res, cur);
            cur -= area(p[i], p[this->nxt(i)], p[j]);
        }
        return res;
    }
};

void solve()

{
    i64 n;
    cin >> n;
    cin >> xc >> yc >> r;
    Convex conv;
    auto  &gra = conv.p;
    gra.resize(n);
    for (int i = 0; i < n; i++) {
        cin >> gra[i].x >> gra[i].y;
    }
    T ans = conv.rotcaliper();
    cout << ans << '\n';
}
```
