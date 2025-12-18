---
title: Almost Aligned
date: 2025-10-18
last_clear: 2025-10-20
tags:
  - QOJ/ICPC/Regionals/Latin_America
  - QOJ/UCUP/2nd/Stage_26_Latin
  - tag/计算几何/CHT
grasp: 3
value: 4
difficulty:
optional_difficulty: Easy-Medium
source: https://qoj.ac/contest/1576/problem/8505
article:
---

```cpp showLineNumbers

using T = double;

struct Line {
    T m, b;

    T intersect_x(const Line &o) const { return (o.b - b) / (m - o.m); }

    T eval(T x) const { return m * x + b; }
};

vector<pair<T, Line>> downConvexHull(vector<Line> lines)
{
    sort(lines.begin(), lines.end(), [](const Line &a, const Line &b) {
        if (a.m != b.m) return a.m < b.m;
        return a.b > b.b;
    });

    vector<Line> hull;
    vector<T>    cross_x;  // 存交点横坐标

    for (auto ln : lines) {
        if (!hull.empty() && ln.m == hull.back().m) continue;  // 斜率相同取截距小的
        while (hull.size() >= 2) {
            T x = ln.intersect_x(hull.back());
            if (x <= cross_x.back()) {
                hull.pop_back();
                cross_x.pop_back();
            } else
                break;
        }
        if (hull.empty()) {
            cross_x.push_back(numeric_limits<T>::lowest());
        } else {
            cross_x.push_back(ln.intersect_x(hull.back()));
        }
        hull.push_back(ln);
    }

    vector<pair<T, Line>> res;
    for (int i = 0; i < hull.size(); ++i) res.push_back({cross_x[i], hull[i]});
    return res;
}

vector<pair<T, Line>> upConvexHull(vector<Line> lines)
{
    sort(lines.begin(), lines.end(), [](const Line &a, const Line &b) {
        if (a.m != b.m) return a.m > b.m;  // 大到小
        return a.b < b.b;                  //
    });

    vector<Line> hull;
    vector<T>    cross_x;  // 存交点横坐标

    for (auto ln : lines) {
        if (!hull.empty() && ln.m == hull.back().m) continue;  // 斜率相同取截距小的
        while (hull.size() >= 2) {
            T x = ln.intersect_x(hull.back());
            if (x <= cross_x.back()) {
                hull.pop_back();
                cross_x.pop_back();
            } else
                break;
        }
        if (hull.empty()) {
            cross_x.push_back(numeric_limits<T>::lowest());
        } else {
            cross_x.push_back(ln.intersect_x(hull.back()));
        }
        hull.push_back(ln);
    }

    vector<pair<T, Line>> res;
    for (int i = 0; i < (int)hull.size(); ++i) res.push_back({cross_x[i], hull[i]});
    return res;
}

void solve()
{
    int n;
    cin >> n;
    vector<array<i64, 4>> points(n);
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < 4; j++) {
            cin >> points[i][j];
        }
    }
    vector<Line> x_lines, y_lines;
    for (int i = 0; i < n; i++) {
        Line lnx = {T(points[i][2]), T(points[i][0])};
        Line lny = {T(points[i][3]), T(points[i][1])};
        x_lines.pb(lnx);
        y_lines.pb(lny);
    }
    auto      upx = downConvexHull(x_lines);
    auto      dnx = upConvexHull(x_lines);
    auto      upy = downConvexHull(y_lines);
    auto      dny = upConvexHull(y_lines);
    vector<T> actiontime;
    for (auto [t, _] : upx) {
        actiontime.pb(t);
    }
    for (auto [t, _] : dnx) {
        actiontime.pb(t);
    }
    for (auto [t, _] : upy) {
        actiontime.pb(t);
    }
    for (auto [t, _] : dny) {
        actiontime.pb(t);
    }
    sort(all(actiontime));
    int ptr = 0;
    while (ptr < actiontime.size() && actiontime[ptr] <= 0) ptr++;
    actiontime = vector<T>(actiontime.begin() + ptr, actiontime.end());
    actiontime.insert(actiontime.begin(), 0);
    actiontime.pb(4e10);
    auto get = [&](T t, const vector<pair<T, Line>> &candi) {
        auto ln = (*prev(upper_bound(all(candi), t, [&](const T &val, const auto &mid) { return val < mid.fi; }))).se;
        return ln;
    };
    T    ans = 8e18;
    int  m   = actiontime.size();
    Line ln_mx_x, ln_mn_x, ln_mx_y, ln_mn_y;
    auto initeval = [&](T tt) constexpr {
        ln_mx_x = get(tt, upx);
        ln_mn_x = get(tt, dnx);
        ln_mx_y = get(tt, upy);
        ln_mn_y = get(tt, dny);
    };
    auto eval = [&](T tt) -> T {
        assert(tt >= 0);
        T mx_x = ln_mx_x.eval(tt), mn_x = ln_mn_x.eval(tt);
        T mx_y = ln_mx_y.eval(tt), mn_y = ln_mn_y.eval(tt);
        T res = (mx_x - mn_x) * (mx_y - mn_y);
        assert(res >= 0);
        ans = min(ans, res);
        return res;
    };
    for (int i = 0; i < m - 1; i++) {
        T left = actiontime[i], right = actiontime[i + 1];
        initeval(left);
        eval(left);
        long double lft = left, rgt = right;
        for (int it = 0; it < 120 || rgt - lft > 1e-14 * lft; ++it) {
            T m1 = lft + (rgt - lft) / 3;
            T m2 = rgt - (rgt - lft) / 3;
            if (eval(m1) < eval(m2)) {
                rgt = m2;
            } else {
                lft = m1;
            }
        }
    }
    cout << fixed << setprecision(18) << ans << '\n';
}
```
