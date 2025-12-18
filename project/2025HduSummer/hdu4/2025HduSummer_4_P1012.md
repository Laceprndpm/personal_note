---
title: 海上的太阳
date: 2025-07-28
last_clear: 2025-08-01
tags:
  - tag/math/asr
grasp: 3
difficulty: 1800
optional_difficulty: Easy-Medium
source: https://acm.hdu.edu.cn/contest/problem?cid=1175&pid=1012
article: https://www.cnblogs.com/pks-t/p/10277958.html
---
# 前言:
模拟+数值积分，算是asr模板题，没有回顾意义
## Adaptive Simpson's Rule
对于曲线积分，因为 $\sqrt{(f'(x)^2 + 1)}$ 没有初等函数表达，难以积分，所以用数值积分法，asr(自适应辛普森方法)其思想是
>利用二次函数来不断**拟合**所求曲线，而所谓的**Adapative(自适应)** 则是用于优化时间复杂度的方法。

$$
\int_l^r f(x)\,dx \approx \frac{f(r) + f(l) + 4 \cdot f(\frac{l + r}{2})}{6}
$$

推导过程参见 [oi-wiki][https://oi-wiki.org/math/numerical/integral/] 
>所谓**自适应**，说的直白点，无非就是**需要多分治几次的地方，多分治几次；不需要的则可以少分治几次**

所以，代码上就可以直接让上次答案与本次答案相减，当差距小于精度要求，就return掉。
关于实现中的15，chatGPT解释为:
>因为单次辛普森误差是整体区间误差的 1 倍，而将区间二分后两次辛普森的误差合计约为原误差的 1/16，因此两者之差近似是误差的 15/16，推得实际误差约为 $\frac{1}{15}(S_L+S_R−S)$ ，故判断条件需写成 15 * eps 才能确保整体误差 ≤ eps。

### 实现:
```c++
myfloat simpson(myfloat l, myfloat r, auto&& f)
{
    myfloat mid = (l + r) / 2;
    return (r - l) * (f(l) + myfloat(4) * f(mid) + f(r)) / 6;  // 辛普森公式
}

myfloat asr(myfloat l, myfloat r, myfloat eps, myfloat ans, int step, auto&& f)
{
    myfloat mid = (l + r) / 2;
    myfloat fl = simpson(l, mid, f), fr = simpson(mid, r, f);
    if (abs(fl + fr - ans) <= 15 * eps && step < 0) return fl + fr + (fl + fr - ans) / 15;  // 足够相似的话就直接返回
    return asr(l, mid, eps / 2, fl, step - 1, f) + asr(mid, r, eps / 2, fr, step - 1, f);   // 否则分割成两段递归求解
}

myfloat calc(myfloat l, myfloat r, myfloat eps, auto&& f)
{
    return asr(l, r, eps, simpson(l, r, f), 12, f); // 至少迭代12次
}
```
### OI-wiki_raw:
```c++
double simpson(double l, double r) {
  double mid = (l + r) / 2;
  return (r - l) * (f(l) + 4 * f(mid) + f(r)) / 6;  // 辛普森公式
}

double asr(double l, double r, double eps, double ans, int step) {
  double mid = (l + r) / 2;
  double fl = simpson(l, mid), fr = simpson(mid, r);
  if (abs(fl + fr - ans) <= 15 * eps && step < 0)
    return fl + fr + (fl + fr - ans) / 15;  // 足够相似的话就直接返回
  return asr(l, mid, eps / 2, fl, step - 1) +
         asr(mid, r, eps / 2, fr, step - 1);  // 否则分割成两段递归求解
}

double calc(double l, double r, double eps) {
  return asr(l, r, eps, simpson(l, r), 12);
}
```

```python
def simpson(l, r):
    mid = (l + r) / 2
    return (r - l) * (f(l) + 4 * f(mid) + f(r)) / 6  # 辛普森公式

def asr(l, r, eps, ans, step):
    mid = (l + r) / 2
    fl = simpson(l, mid)
    fr = simpson(mid, r)
    if abs(fl + fr - ans) <= 15 * eps and step < 0:
        return fl + fr + (fl + fr - ans) / 15  # 足够相似的话就直接返回
    return asr(l, mid, eps / 2, fl, step - 1) + asr(
        mid, r, eps / 2, fr, step - 1
    )  # 否则分割成两段递归求解

def calc(l, r, eps):
    return asr(l, r, eps, simpson(l, r), 12)

```
# 题意:

给定一个定义域为 $[0,+\infty)$ 的非负系数多项式函数

$$
y = a_m x^m + a_{m-1} x^{m-1} + \cdots + a_1 x
$$

表示山坡轮廓。太阳位置为固定点 (X, Y)。

现要在山坡上建造 n 座大楼，第 i 座大楼高度为 $h_i$，居住 $w_i$ 人。大楼底部必须位于山坡曲线上，位置为 $x_i \ge 0$，对应高度为 $y_i = f(x_i)$，宽度忽略不计。

---

## 约束

1. 任意两座大楼底部间欧氏距离至少为两者高度的较大值：

$$
\sqrt{(x_i - x_j)^2 + (y_i - y_j)^2} \ge \max(h_i, h_j), \quad i \neq j
$$

2. 采光条件：任意大楼上的点与太阳连线不得被其他大楼阻挡（允许仅接触端点）。
## 目标

求满足上述条件的 $x_i$ 使得居民从大楼底部**沿山坡**到原点的距离加权和最小：

$$
\sum_{i=1}^n w_i \int_0^{x_i} \sqrt{1 + (f'(x))^2} \, dx
$$

并输出该最小值。

---

答案保证在集合
$$
\{0\} \cup [10^{-9}, 10^{10})
$$
内。
## 输入格式
本题包含多组测试数据。  
第一行输入一个整数 $T$ （$1 \leq T \leq 100$），表示测试数据组数。  
对于每组测试数据，包含 $n + 2$ 行：  
1. 第一行包含四个整数 $n, m, X, Y$，其中 $1 \leq n \leq 6$，$1 \leq m \leq 5$，$-10^5 \leq X \leq -1$，$2 \leq Y \leq 10^5$，分别表示大楼数量、山坡多项式次数、太阳横坐标和纵坐标。  
2. 第二行包含 $m$ 个整数 $a_1, a_2, \ldots, a_m$，满足 $0 \leq a_1, a_2, \ldots, a_{m-1} \leq 100$，$1 \leq a_m \leq 100$，表示山坡多项式函数各系数。  
3. 接下来 $n$ 行，第 $i+2$ 行包含两个整数 $h_i, w_i$，满足 $1 \leq h_i < Y$，$1 \leq w_i \leq 100$，分别表示第 $i$ 座大楼的高度和居民数。  
---

备注：保证满足 $n > 3$ 的测试数据组数不超过五组。
# 正文:
这个数据范围，明显的是 $n!$ 级别的，所以就直接模拟即可，主要考察 $simpson$ 积分。
最终代码:
```c++
void output(long double x)
{
    static char s[25];
    sprintf(s, "%.4Le", x);
    for (int i : {0, 1, 2, 3, 4, 5, 6, 7})
        printf("%c", s[i]);
    printf("%c", s[strlen(s) - 1]);
}

using myfloat         = double;
constexpr myfloat eps = 1e-6;

auto genFunction = [](const vector<i64>& vec) -> auto {
    auto f = [vec](myfloat x) {
        myfloat ans = 0;
        for (int i = sz(vec) - 1; i >= 0; i--) {
            ans = ans * x + vec[i];
        }
        return ans;
    };

    return f;
};

auto qu = [](const vector<i64>& vec) -> auto {
    vector<i64> tmp(vec.size() - 1);
    for (int i = 1; i < sz(vec); i++) {
        tmp[i - 1] = vec[i] * i;
    }
    auto tmpF = genFunction(tmp);
    auto f    = [tmpF](double x) {
        return sqrtl(tmpF(x) * tmpF(x) + 1);
    };
    return f;
};

myfloat simpson(myfloat l, myfloat r, auto&& f)
{
    myfloat mid = (l + r) / 2;
    return (r - l) * (f(l) + myfloat(4) * f(mid) + f(r)) / 6;  // 辛普森公式
}

myfloat asr(myfloat l, myfloat r, myfloat eps, myfloat ans, int step, auto&& f)
{
    myfloat mid = (l + r) / 2;
    myfloat fl = simpson(l, mid, f), fr = simpson(mid, r, f);
    if (abs(fl + fr - ans) <= 15 * eps && step < 0) return fl + fr + (fl + fr - ans) / 15;  // 足够相似的话就直接返回
    return asr(l, mid, eps / 2, fl, step - 1, f) + asr(mid, r, eps / 2, fr, step - 1, f);   // 否则分割成两段递归求解
}

myfloat calc(myfloat l, myfloat r, myfloat eps, auto&& f)
{
    return asr(l, r, eps, simpson(l, r, f), 2, f); // 函数性质良好，减少迭代次数
}

void solve()
{
    int n, m, X, Y;
    cin >> n >> m >> X >> Y;
    vector<i64> ai(m + 1);
    for (int i = 1; i <= m; i++) {
        scanf("%lld", &ai[i]);
    }

    struct Building {
        i64 wi;
        i64 hi;
        int id;

        bool operator<(Building ot)
        {
            return id < ot.id;
        }
    };

    vector<Building> buildings(n);
    for (int i = 0; i < n; i++) {
        scanf("%lld %lld", &buildings[i].hi, &buildings[i].wi);
        buildings[i].id = i;
    }

    sort(all(buildings));
    auto    f      = genFunction(ai);
    auto    quxian = qu(ai);
    myfloat ans    = 1e50;
    do {
        myfloat cur_x   = 0;
        myfloat cur_ans = 0;
        for (int i = 1; i < n; i++) {
            myfloat cur_y       = f(cur_x);
            myfloat cur_y_build = cur_y + buildings[i - 1].hi;
            myfloat l = cur_x, r = 1e18;

            while (r - l > (1e-9)) {
                myfloat mid = (l + r) / 2;
                myfloat res = f(mid);  // (mid, res)
                                       // (res - cur_y_build) / (mid - cur_x) == (cur_y_build - Y) / (cur_x - X)
                if ((res - cur_y_build) * (cur_x - X) > (cur_y_build - Y) * (mid - cur_x)
                    && (cur_y - res) * (cur_y - res) + (mid - cur_x) * (mid - cur_x)
                           > max<myfloat>(buildings[i].hi, buildings[i - 1].hi)
                                 * max<myfloat>(buildings[i].hi, buildings[i - 1].hi))

                {
                    r = mid;
                } else {
                    l = mid;
                }
            }
            cur_x = l;
            cur_ans += calc(0, cur_x, eps, quxian) * buildings[i].wi;
        }
        ans = min<myfloat>(ans, cur_ans);
    } while (next_permutation(all(buildings)));

    output(ans);
    putchar('\n');
}
```
