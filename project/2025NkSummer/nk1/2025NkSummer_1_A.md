---
title: Rectangular Posters
date: 2025-09-12
last_clear: 2025-09-12
tags:
  - tag/math/概率
  - tag/math/积分
  - tag/扫描线
grasp: 0
value: 4
difficulty:
optional_difficulty: Hard-Medium
source: https://ac.nowcoder.com/acm/contest/108298/A
article: file:///home/patchouli/Voile/project/2025NkSummer/attachment/2025NkSummer/1/2025%E7%89%9B%E5%AE%A2%E6%9A%91%E6%9C%9F%E5%A4%9A%E6%A0%A1%E8%AE%AD%E7%BB%83%E8%90%A51%E9%A2%98%E8%A7%A3.pdf
---
# 题意
二维平面上有一块矩形木板，边与坐标轴平行。木板左下角的坐标为 $(0,0)$，右上角的坐标为 $(W,H)$。

有 $n$ 张矩形海报，第 $i$ 张海报的宽度为 $w_i \ (1 \leq w_i < W)$，高度为 $h_i \ (1 \leq h_i < H)$。这些海报随机且独立地完全放置在木板上，不进行旋转或翻转，边与坐标轴平行。更具体地说，第 $i$ 张海报的左下角坐标为 $(x_i,y_i)$，右上角的坐标为 $(x_i+w_i,y_i+h_i)$，其中每个 $x_i$ 都是独立从 $[0,W-w_i]$ 中均匀随机选择的实数，每个 $y_i$ 都是独立从 $[0,H-h_i]$ 中均匀随机选择的实数。

你需要计算 $n$ 张海报覆盖面积的期望，结果对 $10^9+7$ 取模。

---

## Input

第一行包含三个整数 $n \ (1 \leq n \leq 120)$，$W$ 和 $H \ (2 \leq W,H \leq 10^9)$，表示矩形海报的数量、矩形板的宽度和高度。

接下来 $n$ 行，第 $i$ 行包含两个整数 $w_i \ (1 \leq w_i < W)$ 和 $h_i \ (1 \leq h_i < H)$，表示第 $i$ 张矩形海报的宽度和高度。

---

## Output

输出一行包含一个整数，表示 $n$ 张海报覆盖的期望面积对 $10^9+7$ 取模。

可以证明期望始终是一个有理数。此外，在本题的条件下，还可以证明当该值表示为既约分数 $p/q$ 时，$q \not\equiv 0 \pmod{10^9+7}$。因此，存在唯一整数 $r \ (0 \leq r < 10^9+7)$，使得 $p \times r \equiv q \pmod{10^9+7}$。这个 $r$ 即为所求。
# Solution
- 根据期望线性可加，可以考虑每个面元（或者说是点）属于矩形并的概率，然后对面元积分。
- 对每个点再考虑反面，计算一个点不属于矩形并的概率，那么只要考虑这个点不被某个矩形包含的概率，然后将每个矩形的概率乘起来。
- 一个点 $(x,y)$ 被矩形 $(w_i,h_i)$ 包含等价于 $x$ 方向和 $y$ 方向都同向包含，其概率是  
  $\dfrac{\min\{W-w_i,x\}-\max\{0,x-w_i\}}{W-w_i} \cdot \dfrac{\min\{H-h_i,y\}-\max\{0,y-h_i\}}{H-h_i}$，  
  不被矩形包含的概率就是 $1$ 减去这个概率。
---

- 根据每个 $\min$ 和 $\max$ 内的大小关系，可以将 $[0,W]\times[0,H]$ 划分成大约 $4n^2$ 个区域。每个区域里的概率是一个形如 $\prod_{i=1}^n (c_0+c_1x+c_2y+c_3xy)$ 的二元多项式。  
  如果观察到对称性，可以只对 $[0,W/2]\times[0,H/2]$ 进行积分，这样只要处理大约 $n^2$ 个区域。
- 如果对每个区域分别处理，每个乘积项依次乘起来的复杂度是 $O(n^3)$，总复杂度达到 $O(n^5)$，难以通过。

---

- 考虑一行的区域一起处理，从一个区域到相邻区域的时候乘积项会有一些变化，部分乘积项会被移除，然后再新增一些乘积项，可以 $O(n^2)$ 实现单个乘积项的移除和新增，总共只有 $O(n)$ 次移除和新增，这样处理一行是 $O(n^3)$，总复杂度是 $O(n^4)$。
- 实现的时候要注意常数，例如根据 $x$ 和 $y$ 的最高次项，只维护有效范围内的系数矩阵，相比于直接维护完整的 $(n+1)\times(n+1)$ 的系数矩阵，可以分析有 $1/4$ 的常数优化。
```cpp showLineNumbers title="A_0.cpp"
constexpr i64 P = 1e9 + 7;
using Z         = ModInt<P>;

constexpr int N = 120;

Z   f[N + 2][N + 2];
i64 n, W, H;
i64 zero;
Z   con = 1;

void init()
{
    memset(f, 0, sizeof(f));
    zero    = 0;
    con     = 1;
    f[0][0] = 1;
}

pair<Z, Z> calc(i64 WH, i64 wh, i64 x)  // A为{W, H}，c为对应的wi, hi, x为起始点
{
    Z p = 0, q = 0;
    if (x < WH - wh) {
        q += 1;
    } else {
        p += WH - wh;
    }
    if (x < wh) {
        ;
    } else {
        p += wh;
        q -= 1;
    }
    p /= (WH - wh);
    q /= (WH - wh);
    return {p, q};
}

void add(pair<Z, Z> fx, pair<Z, Z> fy)
{
    Z arr[2][2] = {{1 - fx.fi * fy.fi, -fx.fi * fy.se}, {-fx.se * fy.fi, -fx.se * fy.se}};
    if (arr[0][0] == 0) {
        zero++;
        return;
    }
    for (int i = n; i >= 0; i--) {
        for (int j = n; j >= 0; j--) {
            f[i + 1][j + 0] += f[i][j] * arr[1][0];
            f[i + 0][j + 1] += f[i][j] * arr[0][1];
            f[i + 1][j + 1] += f[i][j] * arr[1][1];
        }
    }
    if (arr[0][0] != 1) {
        con *= arr[0][0];
    }
}

void rem(pair<Z, Z> fx, pair<Z, Z> fy)
{
    Z arr[2][2] = {{1 - fx.fi * fy.fi, -fx.fi * fy.se}, {-fx.se * fy.fi, -fx.se * fy.se}};
    if (arr[0][0] == 0) {
        zero--;
        return;
    }
    for (int i = 0; i <= n; i++) {
        for (int j = 0; j <= n; j++) {
            f[i + 1][j + 0] -= f[i][j] * arr[1][0];
            f[i + 0][j + 1] -= f[i][j] * arr[0][1];
            f[i + 1][j + 1] -= f[i][j] * arr[1][1];
        }
    }
    if (arr[0][0] != 1) {
        con /= arr[0][0];
    }
}

void solve()
{
    cin >> n >> W >> H;

    W *= 2, H *= 2;
    vector<i64> wi(n), hi(n);
    for (int i = 0; i < n; i++) {
        cin >> wi[i] >> hi[i];
        wi[i] *= 2;
        hi[i] *= 2;
    }
    vector<i64> acx{0, W / 2}, acy{0, H / 2};
    for (int i = 0; i < n; i++) {
        acx.pb(min(wi[i], W - wi[i]));
        acy.pb(min(hi[i], H - hi[i]));
    }
    sor(acx), sor(acy);
    acx.erase(unique(all(acx)), acx.end());
    acy.erase(unique(all(acy)), acy.end());
    const int           lenx = sz(acx), leny = sz(acy);
    vector<vector<int>> acid(leny);
    for (int i = 0; i < n; i++) {
        const int id = lower_bound(all(acy), min(hi[i], H - hi[i])) - acy.begin();
        acid[id].pb(i);
    }
    Z ans = 0;
    for (int s = 0; s < sz(acx) - 1; s++) {
        const i64 x0 = acx[s], x1 = acx[s + 1];
        init();
        for (int i = 0; i < n; i++) {
            const auto a = calc(W, wi[i], x0);  // 扫的x,所以x可以确定
            const auto b = calc(H, hi[i], 0);   // 从0开始
            add(a, b);
        }
        for (int t = 0; t < sz(acy) - 1; t++) {
            const i64 y0 = acy[t], y1 = acy[t + 1];
            for (const int i : acid[t]) {
                const auto a  = calc(W, wi[i], x0);
                const auto b  = calc(H, hi[i], 0);
                const auto bb = calc(H, hi[i], y0);
                rem(a, b);
                add(a, bb);
            }
            vector<Z> inteX(n + 1), inteY(n + 1);
            {
                Z a = 1, b = 1;
                for (int k = 0; k <= n; k++) {
                    a *= x1, b *= x0;
                    inteX[k] = (b - a) / (k + 1);
                }
            }
            {
                Z a = 1, b = 1;
                for (int k = 0; k <= n; k++) {
                    a *= y1, b *= y0;
                    inteY[k] = (b - a) / (k + 1);
                }
            }
            if (zero) continue;
            Z sum = 0;
            for (int k = 0; k <= n; ++k)
                for (int l = 0; l <= n; ++l) sum += f[k][l] * inteX[k] * inteY[l];
            ans += sum * con;
        }
    }
    ans = Z(W / 2) * (H / 2) - ans;

    cout << ans << '\n';
}
```
