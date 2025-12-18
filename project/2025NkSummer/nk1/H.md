---
title: Symmetry Intervals 2
date: 2025-07-16
last_clear: 2025-07-16
tags:
  - tag/bitmasks
  - tag/dp/状压dp
grasp: 2
difficulty: 2200
source: https://ac.nowcoder.com/acm/contest/108298/H
article:
---
# 题意
给定一个长度为 $n$ 的二进制字符串 $S$，你需要回答 $q$ 个查询，每个查询有以下两种类型之一：
1. 给定两个整数 $l, r$（$1 \le l \le r \le n$），将区间 $[l, r]$ 内的每个二进制位 $S_i$ 翻转。
2. 给定三个整数 $l, a, b$（$1 \le l \le n, 1 \le a, b \le n-l+1$），你需要求出区间 $[1, l]$ 内有多少个区间 $[u, v]$（$1 \le u \le v \le l$）满足  
   $S_{a+x-1} = S_{b+x-1}$ 对每个整数 $x \in [u, v]$  
   这样的区间称为对称区间（symmetry intervals）。
# Solution
首先可以很容易写出朴素做法，即
1. 对于操作一，只要懒标记前缀和，操作二时统一处理即可。那么该复杂度为$O(2500 n)$ 。
2. 对于操作二，无论是直接计数还是直接区间合并，都比较直观，复杂度也为$O(2500n)$ 。
可以注意到这两个操作都有点超时，因此可以用 bitset 优化，本质是分块。
操作一，将前缀和只统计覆盖整块的，而不足整块则直接标记，复杂度可以降到
$O(64q + 2500 n / 64)$。
操作二，则可以设01状态为匹配/不匹配后直接dp，按16bit/块处理后，区间合并。时间复杂度降到$O(2500n / 16)$ 约为 `156'250'000`。
比较关键的是 bitset 的写法，bitset的shift操作可以优化到$n / 32$ ，其中32是块长的 $1/2$。
```c++
// BEGIN: main.cpp
#line 1 "main.cpp"
// BEGIN: my_template.hpp
#line 1 "my_template.hpp"
#if defined(LOCAL)
#include <my_template_compiled.hpp>
#else

// https://codeforces.com/blog/entry/96344
// https://codeforces.com/blog/entry/126772?#comment-1154880
#include <bits/allocator.h>
#pragma GCC optimize("Ofast,unroll-loops")
#pragma GCC target("avx2,popcnt")
#include <bits/stdc++.h>

using namespace std;

using ll = long long;
using u8 = uint8_t;
using u16 = uint16_t;
using u32 = uint32_t;
using u64 = uint64_t;
using i128 = __int128;
using u128 = unsigned __int128;
using f128 = __float128;

template <class T>
constexpr T infty = 0;
template <>
constexpr int infty<int> = 1'010'000'000;
template <>
constexpr ll infty<ll> = 2'020'000'000'000'000'000;
template <>
constexpr u32 infty<u32> = infty<int>;
template <>
constexpr u64 infty<u64> = infty<ll>;
template <>
constexpr i128 infty<i128> = i128(infty<ll>) * 2'000'000'000'000'000'000;
template <>
constexpr double infty<double> = infty<ll>;
template <>
constexpr long double infty<long double> = infty<ll>;

using pi = pair<ll, ll>;
using vi = vector<ll>;
template <class T>
using vc = vector<T>;
template <class T>
using vvc = vector<vc<T>>;
template <class T>
using vvvc = vector<vvc<T>>;
template <class T>
using vvvvc = vector<vvvc<T>>;
template <class T>
using vvvvvc = vector<vvvvc<T>>;
template <class T>
using pq_max = priority_queue<T>;
template <class T>
using pq_min = priority_queue<T, vector<T>, greater<T>>;

#define vv(type, name, h, ...) \
  vector<vector<type>> name(h, vector<type>(__VA_ARGS__))
#define vvv(type, name, h, w, ...)   \
  vector<vector<vector<type>>> name( \
      h, vector<vector<type>>(w, vector<type>(__VA_ARGS__)))
#define vvvv(type, name, a, b, c, ...)       \
  vector<vector<vector<vector<type>>>> name( \
      a, vector<vector<vector<type>>>(       \
             b, vector<vector<type>>(c, vector<type>(__VA_ARGS__))))

// https://trap.jp/post/1224/
#define FOR1(a) for (ll _ = 0; _ < ll(a); ++_)
#define FOR2(i, a) for (ll i = 0; i < ll(a); ++i)
#define FOR3(i, a, b) for (ll i = a; i < ll(b); ++i)
#define FOR4(i, a, b, c) for (ll i = a; i < ll(b); i += (c))
#define FOR1_R(a) for (ll i = (a) - 1; i >= ll(0); --i)
#define FOR2_R(i, a) for (ll i = (a) - 1; i >= ll(0); --i)
#define FOR3_R(i, a, b) for (ll i = (b) - 1; i >= ll(a); --i)
#define overload4(a, b, c, d, e, ...) e
#define overload3(a, b, c, d, ...) d
#define FOR(...) overload4(__VA_ARGS__, FOR4, FOR3, FOR2, FOR1)(__VA_ARGS__)
#define FOR_R(...) overload3(__VA_ARGS__, FOR3_R, FOR2_R, FOR1_R)(__VA_ARGS__)

#define all(x) x.begin(), x.end()
#define len(x) ll(x.size())
#define elif else if

#define eb emplace_back
#define mp make_pair
#define mt make_tuple
#define fi first
#define se second

#define stoi stoll

int popcnt(int x) { return __builtin_popcount(x); }
int popcnt(u32 x) { return __builtin_popcount(x); }
int popcnt(ll x) { return __builtin_popcountll(x); }
int popcnt(u64 x) { return __builtin_popcountll(x); }
int popcnt_sgn(int x) { return (__builtin_parity(unsigned(x)) & 1 ? -1 : 1); }
int popcnt_sgn(u32 x) { return (__builtin_parity(x) & 1 ? -1 : 1); }
int popcnt_sgn(ll x) { return (__builtin_parityll(x) & 1 ? -1 : 1); }
int popcnt_sgn(u64 x) { return (__builtin_parityll(x) & 1 ? -1 : 1); }
// (0, 1, 2, 3, 4) -> (-1, 0, 1, 1, 2)
int topbit(int x) { return (x == 0 ? -1 : 31 - __builtin_clz(x)); }
int topbit(u32 x) { return (x == 0 ? -1 : 31 - __builtin_clz(x)); }
int topbit(ll x) { return (x == 0 ? -1 : 63 - __builtin_clzll(x)); }
int topbit(u64 x) { return (x == 0 ? -1 : 63 - __builtin_clzll(x)); }
// (0, 1, 2, 3, 4) -> (-1, 0, 1, 0, 2)
int lowbit(int x) { return (x == 0 ? -1 : __builtin_ctz(x)); }
int lowbit(u32 x) { return (x == 0 ? -1 : __builtin_ctz(x)); }
int lowbit(ll x) { return (x == 0 ? -1 : __builtin_ctzll(x)); }
int lowbit(u64 x) { return (x == 0 ? -1 : __builtin_ctzll(x)); }

template <typename T>
T kth_bit(int k) {
  return T(1) << k;
}
template <typename T>
bool has_kth_bit(T x, int k) {
  return x >> k & 1;
}

template <typename UINT>
struct all_bit {
  struct iter {
    UINT s;
    iter(UINT s) : s(s) {}
    int operator*() const { return lowbit(s); }
    iter &operator++() {
      s &= s - 1;
      return *this;
    }
    bool operator!=(const iter) const { return s != 0; }
  };
  UINT s;
  all_bit(UINT s) : s(s) {}
  iter begin() const { return iter(s); }
  iter end() const { return iter(0); }
};

template <typename UINT>
struct all_subset {
  static_assert(is_unsigned<UINT>::value);
  struct iter {
    UINT s, t;
    bool ed;
    iter(UINT s) : s(s), t(s), ed(0) {}
    int operator*() const { return s ^ t; }
    iter &operator++() {
      (t == 0 ? ed = 1 : t = (t - 1) & s);
      return *this;
    }
    bool operator!=(const iter) const { return !ed; }
  };
  UINT s;
  all_subset(UINT s) : s(s) {}
  iter begin() const { return iter(s); }
  iter end() const { return iter(0); }
};

template <typename T>
T floor(T a, T b) {
  return a / b - (a % b && (a ^ b) < 0);
}
template <typename T>
T ceil(T x, T y) {
  return floor(x + y - 1, y);
}
template <typename T>
T bmod(T x, T y) {
  return x - y * floor(x, y);
}
template <typename T>
pair<T, T> divmod(T x, T y) {
  T q = floor(x, y);
  return {q, x - q * y};
}

template <typename T, typename U>
T SUM(const U &A) {
  return std::accumulate(A.begin(), A.end(), T{});
}

#define MIN(v) *min_element(all(v))
#define MAX(v) *max_element(all(v))
#define LB(c, x) distance((c).begin(), lower_bound(all(c), (x)))
#define UB(c, x) distance((c).begin(), upper_bound(all(c), (x)))
#define UNIQUE(x) \
  sort(all(x)), x.erase(unique(all(x)), x.end()), x.shrink_to_fit()

template <typename T>
T POP(deque<T> &que) {
  T a = que.front();
  que.pop_front();
  return a;
}
template <typename T>
T POP(pq_min<T> &que) {
  T a = que.top();
  que.pop();
  return a;
}
template <typename T>
T POP(pq_max<T> &que) {
  T a = que.top();
  que.pop();
  return a;
}
template <typename T>
T POP(vc<T> &que) {
  T a = que.back();
  que.pop_back();
  return a;
}

template <typename F>
ll binary_search(F check, ll ok, ll ng, bool check_ok = true) {
  if (check_ok) assert(check(ok));
  while (abs(ok - ng) > 1) {
    auto x = (ng + ok) / 2;
    (check(x) ? ok : ng) = x;
  }
  return ok;
}
template <typename F>
double binary_search_real(F check, double ok, double ng, int iter = 100) {
  FOR(iter) {
    double x = (ok + ng) / 2;
    (check(x) ? ok : ng) = x;
  }
  return (ok + ng) / 2;
}

template <class T, class S>
inline bool chmax(T &a, const S &b) {
  return (a < b ? a = b, 1 : 0);
}
template <class T, class S>
inline bool chmin(T &a, const S &b) {
  return (a > b ? a = b, 1 : 0);
}

// ? は -1
vc<int> s_to_vi(const string &S, char first_char) {
  vc<int> A(S.size());
  FOR(i, S.size()) { A[i] = (S[i] != '?' ? S[i] - first_char : -1); }
  return A;
}

template <typename T, typename U>
vector<T> cumsum(vector<U> &A, int off = 1) {
  int N = A.size();
  vector<T> B(N + 1);
  FOR(i, N) { B[i + 1] = B[i] + A[i]; }
  if (off == 0) B.erase(B.begin());
  return B;
}

// stable sort
template <typename T>
vector<int> argsort(const vector<T> &A) {
  vector<int> ids(len(A));
  iota(all(ids), 0);
  sort(all(ids),
       [&](int i, int j) { return (A[i] == A[j] ? i < j : A[i] < A[j]); });
  return ids;
}

// A[I[0]], A[I[1]], ...
template <typename T>
vc<T> rearrange(const vc<T> &A, const vc<int> &I) {
  vc<T> B(len(I));
  FOR(i, len(I)) B[i] = A[I[i]];
  return B;
}

template <typename T, typename... Vectors>
void concat(vc<T> &first, const Vectors &...others) {
  vc<T> &res = first;
  (res.insert(res.end(), others.begin(), others.end()), ...);
}
#endif
// END: my_template.hpp
#line 2 "main.cpp"
// BEGIN: other/io.hpp
#line 1 "other/io.hpp"
#define FASTIO
#include <unistd.h>

// https://judge.yosupo.jp/submission/21623
namespace fastio {
static constexpr uint32_t SZ = 1 << 17;
char ibuf[SZ];
char obuf[SZ];
char out[100];
// pointer of ibuf, obuf
uint32_t pil = 0, pir = 0, por = 0;

struct Pre {
  char num[10000][4];
  constexpr Pre() : num() {
    for (int i = 0; i < 10000; i++) {
      int n = i;
      for (int j = 3; j >= 0; j--) {
        num[i][j] = n % 10 | '0';
        n /= 10;
      }
    }
  }
} constexpr pre;

inline void load() {
  memcpy(ibuf, ibuf + pil, pir - pil);
  pir = pir - pil + fread(ibuf + pir - pil, 1, SZ - pir + pil, stdin);
  pil = 0;
  if (pir < SZ) ibuf[pir++] = '\n';
}

inline void flush() {
  fwrite(obuf, 1, por, stdout);
  por = 0;
}

void rd(char &c) {
  do {
    if (pil + 1 > pir) load();
    c = ibuf[pil++];
  } while (isspace(c));
}

void rd(string &x) {
  x.clear();
  char c;
  do {
    if (pil + 1 > pir) load();
    c = ibuf[pil++];
  } while (isspace(c));
  do {
    x += c;
    if (pil == pir) load();
    c = ibuf[pil++];
  } while (!isspace(c));
}

template <typename T>
void rd_real(T &x) {
  string s;
  rd(s);
  x = stod(s);
}

template <typename T>
void rd_integer(T &x) {
  if (pil + 100 > pir) load();
  char c;
  do
    c = ibuf[pil++];
  while (c < '-');
  bool minus = 0;
  if constexpr (is_signed<T>::value || is_same_v<T, i128>) {
    if (c == '-') { minus = 1, c = ibuf[pil++]; }
  }
  x = 0;
  while ('0' <= c) { x = x * 10 + (c & 15), c = ibuf[pil++]; }
  if constexpr (is_signed<T>::value || is_same_v<T, i128>) {
    if (minus) x = -x;
  }
}

void rd(int &x) { rd_integer(x); }
void rd(ll &x) { rd_integer(x); }
void rd(i128 &x) { rd_integer(x); }
void rd(u32 &x) { rd_integer(x); }
void rd(u64 &x) { rd_integer(x); }
void rd(u128 &x) { rd_integer(x); }
void rd(double &x) { rd_real(x); }
void rd(long double &x) { rd_real(x); }
void rd(f128 &x) { rd_real(x); }

template <class T, class U>
void rd(pair<T, U> &p) {
  return rd(p.first), rd(p.second);
}
template <size_t N = 0, typename T>
void rd_tuple(T &t) {
  if constexpr (N < std::tuple_size<T>::value) {
    auto &x = std::get<N>(t);
    rd(x);
    rd_tuple<N + 1>(t);
  }
}
template <class... T>
void rd(tuple<T...> &tpl) {
  rd_tuple(tpl);
}

template <size_t N = 0, typename T>
void rd(array<T, N> &x) {
  for (auto &d: x) rd(d);
}
template <class T>
void rd(vc<T> &x) {
  for (auto &d: x) rd(d);
}

void read() {}
template <class H, class... T>
void read(H &h, T &... t) {
  rd(h), read(t...);
}

void wt(const char c) {
  if (por == SZ) flush();
  obuf[por++] = c;
}
void wt(const string s) {
  for (char c: s) wt(c);
}
void wt(const char *s) {
  size_t len = strlen(s);
  for (size_t i = 0; i < len; i++) wt(s[i]);
}

template <typename T>
void wt_integer(T x) {
  if (por > SZ - 100) flush();
  if (x < 0) { obuf[por++] = '-', x = -x; }
  int outi;
  for (outi = 96; x >= 10000; outi -= 4) {
    memcpy(out + outi, pre.num[x % 10000], 4);
    x /= 10000;
  }
  if (x >= 1000) {
    memcpy(obuf + por, pre.num[x], 4);
    por += 4;
  } else if (x >= 100) {
    memcpy(obuf + por, pre.num[x] + 1, 3);
    por += 3;
  } else if (x >= 10) {
    int q = (x * 103) >> 10;
    obuf[por] = q | '0';
    obuf[por + 1] = (x - q * 10) | '0';
    por += 2;
  } else
    obuf[por++] = x | '0';
  memcpy(obuf + por, out + outi + 4, 96 - outi);
  por += 96 - outi;
}

template <typename T>
void wt_real(T x) {
  ostringstream oss;
  oss << fixed << setprecision(15) << double(x);
  string s = oss.str();
  wt(s);
}

void wt(int x) { wt_integer(x); }
void wt(ll x) { wt_integer(x); }
void wt(i128 x) { wt_integer(x); }
void wt(u32 x) { wt_integer(x); }
void wt(u64 x) { wt_integer(x); }
void wt(u128 x) { wt_integer(x); }
void wt(double x) { wt_real(x); }
void wt(long double x) { wt_real(x); }
void wt(f128 x) { wt_real(x); }

template <class T, class U>
void wt(const pair<T, U> val) {
  wt(val.first);
  wt(' ');
  wt(val.second);
}
template <size_t N = 0, typename T>
void wt_tuple(const T t) {
  if constexpr (N < std::tuple_size<T>::value) {
    if constexpr (N > 0) { wt(' '); }
    const auto x = std::get<N>(t);
    wt(x);
    wt_tuple<N + 1>(t);
  }
}
template <class... T>
void wt(tuple<T...> tpl) {
  wt_tuple(tpl);
}
template <class T, size_t S>
void wt(const array<T, S> val) {
  auto n = val.size();
  for (size_t i = 0; i < n; i++) {
    if (i) wt(' ');
    wt(val[i]);
  }
}
template <class T>
void wt(const vector<T> val) {
  auto n = val.size();
  for (size_t i = 0; i < n; i++) {
    if (i) wt(' ');
    wt(val[i]);
  }
}

void print() { wt('\n'); }
template <class Head, class... Tail>
void print(Head &&head, Tail &&... tail) {
  wt(head);
  if (sizeof...(Tail)) wt(' ');
  print(forward<Tail>(tail)...);
}

// gcc expansion. called automaticall after main.
void __attribute__((destructor)) _d() { flush(); }
} // namespace fastio
using fastio::read;
using fastio::print;
using fastio::flush;

#if defined(LOCAL)
#define SHOW(...) SHOW_IMPL(__VA_ARGS__, SHOW6, SHOW5, SHOW4, SHOW3, SHOW2, SHOW1)(__VA_ARGS__)
#define SHOW_IMPL(_1, _2, _3, _4, _5, _6, NAME, ...) NAME
#define SHOW1(x) print(#x, "=", (x)), flush()
#define SHOW2(x, y) print(#x, "=", (x), #y, "=", (y)), flush()
#define SHOW3(x, y, z) print(#x, "=", (x), #y, "=", (y), #z, "=", (z)), flush()
#define SHOW4(x, y, z, w) print(#x, "=", (x), #y, "=", (y), #z, "=", (z), #w, "=", (w)), flush()
#define SHOW5(x, y, z, w, v) print(#x, "=", (x), #y, "=", (y), #z, "=", (z), #w, "=", (w), #v, "=", (v)), flush()
#define SHOW6(x, y, z, w, v, u) print(#x, "=", (x), #y, "=", (y), #z, "=", (z), #w, "=", (w), #v, "=", (v), #u, "=", (u)), flush()
#else
#define SHOW(...)
#endif

#define INT(...)   \
  int __VA_ARGS__; \
  read(__VA_ARGS__)
#define LL(...)   \
  ll __VA_ARGS__; \
  read(__VA_ARGS__)
#define U32(...)   \
  u32 __VA_ARGS__; \
  read(__VA_ARGS__)
#define U64(...)   \
  u64 __VA_ARGS__; \
  read(__VA_ARGS__)
#define STR(...)      \
  string __VA_ARGS__; \
  read(__VA_ARGS__)
#define CHAR(...)   \
  char __VA_ARGS__; \
  read(__VA_ARGS__)
#define DBL(...)      \
  double __VA_ARGS__; \
  read(__VA_ARGS__)

#define VEC(type, name, size) \
  vector<type> name(size);    \
  read(name)
#define VV(type, name, h, w)                     \
  vector<vector<type>> name(h, vector<type>(w)); \
  read(name)

void YES(bool t = 1) { print(t ? "YES" : "NO"); }
void NO(bool t = 1) { YES(!t); }
void Yes(bool t = 1) { print(t ? "Yes" : "No"); }
void No(bool t = 1) { Yes(!t); }
void yes(bool t = 1) { print(t ? "yes" : "no"); }
void no(bool t = 1) { yes(!t); }
void YA(bool t = 1) { print(t ? "YA" : "TIDAK"); }
void TIDAK(bool t = 1) { YA(!t); }
// END: other/io.hpp
#line 3 "main.cpp"

// BEGIN: ds/my_bitset.hpp
#line 1 "ds/my_bitset.hpp"

// https://codeforces.com/contest/914/problem/F
// https://yukicoder.me/problems/no/142
// わずかに普通の bitset より遅いときもあるようだが，
// 固定長にしたくないときや slice 操作が必要なときに使う
struct My_Bitset {
  using T = My_Bitset;
  int N;
  vc<u64> dat;

  // x で埋める
  My_Bitset(int N = 0, int x = 0) : N(N) {
    assert(x == 0 || x == 1);
    u64 v = (x == 0 ? 0 : -1);
    dat.assign((N + 63) >> 6, v);
    if (N) dat.back() >>= (64 * len(dat) - N);
  }

  int size() { return N; }

  void resize(int size) {
    dat.resize((size + 63) >> 6);
    int remainingBits = size & 63;
    if (remainingBits != 0) {
      u64 mask = (u64(1) << remainingBits) - 1;
      dat.back() &= mask;
    }
    N = size;
  }

  void append(int idx, bool b) {
    assert(N == idx);
    resize(idx + 1), (*this)[idx] = b;
  }

  static T from_string(string &S) {
    int N = len(S);
    T ANS(N);
    FOR(i, N) ANS[i] = (S[i] == '1');
    return ANS;
  }

  // thanks to chatgpt!
  class Proxy {
  public:
    Proxy(vc<u64> &d, int i) : dat(d), index(i) {}
    operator bool() const { return (dat[index >> 6] >> (index & 63)) & 1; }

    Proxy &operator=(u64 value) {
      dat[index >> 6] &= ~(u64(1) << (index & 63));
      dat[index >> 6] |= (value & 1) << (index & 63);
      return *this;
    }
    void flip() {
      dat[index >> 6] ^= (u64(1) << (index & 63)); // XOR to flip the bit
    }

  private:
    vc<u64> &dat;
    int index;
  };

  Proxy operator[](int i) { return Proxy(dat, i); }

  bool operator==(const T &p) {
    assert(N == p.N);
    FOR(i, len(dat)) if (dat[i] != p.dat[i]) return false;
    return true;
  }

  T &operator&=(const T &p) {
    assert(N == p.N);
    FOR(i, len(dat)) dat[i] &= p.dat[i];
    return *this;
  }
  T &operator|=(const T &p) {
    assert(N == p.N);
    FOR(i, len(dat)) dat[i] |= p.dat[i];
    return *this;
  }
  T &operator^=(const T &p) {
    assert(N == p.N);
    FOR(i, len(dat)) dat[i] ^= p.dat[i];
    return *this;
  }
  T operator&(const T &p) const { return T(*this) &= p; }
  T operator|(const T &p) const { return T(*this) |= p; }
  T operator^(const T &p) const { return T(*this) ^= p; }
  T operator~() const {
    T p = (*this);
    p.flip_range(0, N);
    return p;
  }

  void set_minus_inplace(T &other) {
    assert(N == other.N);
    FOR(i, len(dat)) dat[i] = dat[i] & (~other.dat[i]);
  }

  T set_minus(T other) {
    assert(N == other.N);
    FOR(i, len(dat)) other.dat[i] = dat[i] & (~other.dat[i]);
    return other;
  }

  int count() {
    int ans = 0;
    for (u64 val: dat) ans += popcnt(val);
    return ans;
  }

  int dot(T &p) {
    assert(N == p.N);
    int ans = 0;
    FOR(i, len(dat)) ans += popcnt(dat[i] & p.dat[i]);
    return ans;
  }

  int next(int i) {
    chmax(i, 0);
    if (i >= N) return N;
    int k = i >> 6;
    {
      u64 x = dat[k];
      int s = i & 63;
      x = (x >> s) << s;
      if (x) return (k << 6) | lowbit(x);
    }
    FOR(idx, k + 1, len(dat)) {
      if (dat[idx] == 0) continue;
      return (idx << 6) | lowbit(dat[idx]);
    }
    return N;
  }

  int prev(int i) {
    chmin(i, N - 1);
    if (i <= -1) return -1;
    int k = i >> 6;
    if ((i & 63) < 63) {
      u64 x = dat[k];
      x &= (u64(1) << ((i & 63) + 1)) - 1;
      if (x) return (k << 6) | topbit(x);
      --k;
    }
    FOR_R(idx, k + 1) {
      if (dat[idx] == 0) continue;
      return (idx << 6) | topbit(dat[idx]);
    }
    return -1;
  }

  My_Bitset range(int L, int R) {
    assert(L <= R);
    My_Bitset p(R - L);
    int rm = (R - L) & 63;
    FOR(rm) {
      p[R - L - 1] = bool((*this)[R - 1]);
      --R;
    }
    int n = (R - L) >> 6;
    int hi = L & 63;
    int lo = 64 - hi;
    int s = L >> 6;
    if (hi == 0) {
      FOR(i, n) { p.dat[i] ^= dat[s + i]; }
    } else {
      FOR(i, n) { p.dat[i] ^= (dat[s + i] >> hi) ^ (dat[s + i + 1] << lo); }
    }
    return p;
  }

  My_Bitset slice(int L, int R) { return range(L, R); }

  int count_range(int L, int R) {
    assert(L <= R);
    int cnt = 0;
    while ((L < R) && (L & 63)) cnt += (*this)[L++];
    while ((L < R) && (R & 63)) cnt += (*this)[--R];
    int l = L >> 6, r = R >> 6;
    FOR(i, l, r) cnt += popcnt(dat[i]);
    return cnt;
  }

  // [L,R) に p を代入
  void assign_to_range(int L, int R, My_Bitset &p) {
    assert(p.N == R - L);
    int a = 0, b = p.N;
    while (L < R && (L & 63)) { (*this)[L++] = bool(p[a++]); }
    while (L < R && (R & 63)) { (*this)[--R] = bool(p[--b]); }
    // p[a:b] を [L:R] に
    int l = L >> 6, r = R >> 6;
    int s = a >> 6, t = b >> t;
    int n = r - l;
    if (!(a & 63)) {
      FOR(i, n) dat[l + i] = p.dat[s + i];
    } else {
      int hi = a & 63;
      int lo = 64 - hi;
      FOR(i, n) dat[l + i] = (p.dat[s + i] >> hi) | (p.dat[1 + s + i] << lo);
    }
  }

  // [L,R) に p を xor
  void xor_to_range(int L, int R, My_Bitset &p) {
    assert(p.N == R - L);
    int a = 0, b = p.N;
    while (L < R && (L & 63)) {
      dat[L >> 6] ^= u64(p[a]) << (L & 63);
      ++a, ++L;
    }
    while (L < R && (R & 63)) {
      --b, --R;
      dat[R >> 6] ^= u64(p[b]) << (R & 63);
    }
    // p[a:b] を [L:R] に
    int l = L >> 6, r = R >> 6;
    int s = a >> 6, t = b >> t;
    int n = r - l;
    if (!(a & 63)) {
      FOR(i, n) dat[l + i] ^= p.dat[s + i];
    } else {
      int hi = a & 63;
      int lo = 64 - hi;
      FOR(i, n) dat[l + i] ^= (p.dat[s + i] >> hi) | (p.dat[1 + s + i] << lo);
    }
  }

  // 行列基本変形で使うやつ
  // p は [i:N) にしかないとして p を xor する
  void xor_suffix(int i, My_Bitset &p) {
    assert(N == p.N && 0 <= i && i < N);
    FOR(k, i / 64, len(dat)) { dat[k] ^= p.dat[k]; }
  }

  // [L,R) に p を and
  void and_to_range(int L, int R, My_Bitset &p) {
    assert(p.N == R - L);
    int a = 0, b = p.N;
    while (L < R && (L & 63)) {
      if (!p[a]) (*this)[L] = 0;
      a++, L++;
    }
    while (L < R && (R & 63)) {
      --b, --R;
      if (!p[b]) (*this)[R] = 0;
    }
    // p[a:b] を [L:R] に
    int l = L >> 6, r = R >> 6;
    int s = a >> 6, t = b >> t;
    int n = r - l;
    if (!(a & 63)) {
      FOR(i, n) dat[l + i] &= p.dat[s + i];
    } else {
      int hi = a & 63;
      int lo = 64 - hi;
      FOR(i, n) dat[l + i] &= (p.dat[s + i] >> hi) | (p.dat[1 + s + i] << lo);
    }
  }

  // [L,R) に p を or
  void or_to_range(int L, int R, My_Bitset &p) {
    assert(p.N == R - L);
    int a = 0, b = p.N;
    while (L < R && (L & 63)) {
      dat[L >> 6] |= u64(p[a]) << (L & 63);
      ++a, ++L;
    }
    while (L < R && (R & 63)) {
      --b, --R;
      dat[R >> 6] |= u64(p[b]) << (R & 63);
    }
    // p[a:b] を [L:R] に
    int l = L >> 6, r = R >> 6;
    int s = a >> 6, t = b >> t;
    int n = r - l;
    if (!(a & 63)) {
      FOR(i, n) dat[l + i] |= p.dat[s + i];
    } else {
      int hi = a & 63;
      int lo = 64 - hi;
      FOR(i, n) dat[l + i] |= (p.dat[s + i] >> hi) | (p.dat[1 + s + i] << lo);
    }
  }
  // 行列基本変形で使うやつ
  // p は [i:N) にしかないとして p を or する
  void or_suffix(int i, My_Bitset &p) {
    assert(N == p.N && 0 <= i && i < N);
    FOR(k, i / 64, len(dat)) { dat[k] |= p.dat[k]; }
  }

  // [L,R) を 1 に変更
  void set_range(int L, int R) {
    while (L < R && (L & 63)) { set(L++); }
    while (L < R && (R & 63)) { set(--R); }
    FOR(i, L >> 6, R >> 6) dat[i] = u64(-1);
  }

  // [L,R) を 1 に変更
  void reset_range(int L, int R) {
    while (L < R && (L & 63)) { reset(L++); }
    while (L < R && (R & 63)) { reset(--R); }
    FOR(i, L >> 6, R >> 6) dat[i] = u64(0);
  }

  // [L,R) を flip
  void flip_range(int L, int R) {
    while (L < R && (L & 63)) { flip(L++); }
    while (L < R && (R & 63)) { flip(--R); }
    FOR(i, L >> 6, R >> 6) dat[i] ^= u64(-1);
  }

  // bitset に仕様を合わせる
  void set(int i) { (*this)[i] = 1; }
  void reset(int i) { (*this)[i] = 0; }
  void flip(int i) { (*this)[i].flip(); }
  void set() {
    fill(all(dat), u64(-1));
    resize(N);
  }
  void reset() { fill(all(dat), 0); }
  void flip() {
    FOR(i, len(dat) - 1) { dat[i] = u64(-1) ^ dat[i]; }
    int i = len(dat) - 1;
    FOR(k, 64) {
      if (64 * i + k >= size()) break;
      flip(64 * i + k);
    }
  }
  bool any() {
    FOR(i, len(dat)) {
      if (dat[i]) return true;
    }
    return false;
  }

  bool ALL() {
    dat.resize((N + 63) >> 6);
    int r = N & 63;
    if (r != 0) {
      u64 mask = (u64(1) << r) - 1;
      if (dat.back() != mask) return 0;
    }
    for (int i = 0; i < N / 64; ++i)
      if (dat[i] != u64(-1)) return false;
    return true;
  }
  // bs[i]==true であるような i 全体
  vc<int> collect_idx() {
    vc<int> I;
    FOR(i, N) if ((*this)[i]) I.eb(i);
    return I;
  }

  bool is_subset(T &other) {
    assert(len(other) == N);
    FOR(i, len(dat)) {
      u64 a = dat[i], b = other.dat[i];
      if ((a & b) != a) return false;
    }
    return true;
  }

  int _Find_first() { return next(0); }
  int _Find_next(int p) { return next(p + 1); }

  template <typename F>
  void enumerate(int L, int R, F f) {
    if (L >= size()) return;
    int p = ((*this)[L] ? L : _Find_next(L));
    while (p < R) {
      f(p);
      p = _Find_next(p);
    }
  }

  static string TO_STR[256];
  string to_string() const {
    if (TO_STR[0].empty()) precompute();
    string S;
    for (auto &x: dat) { FOR(i, 8) S += TO_STR[(x >> (8 * i) & 255)]; }
    S.resize(N);
    return S;
  }

  static void precompute() {
    FOR(s, 256) {
      string x;
      FOR(i, 8) x += '0' + (s >> i & 1);
      TO_STR[s] = x;
    }
  }
};
string My_Bitset::TO_STR[256];// END: ds/my_bitset.hpp
#line 5 "main.cpp"

struct T {
  ll a, b, c, d;
};

struct Mono {
  using value_type = T;
  using X = value_type;
  static X op_R(X L, X R) {
    swap(L, R);
    return T{L.a + R.a * L.c, L.b + R.a * L.d + R.b, R.c * L.c,
             R.c * L.d + R.d};
  }
  static constexpr X unit() { return T{0, 0, 1, 0}; }
  static constexpr bool commute = 0;
};

T dat[1 << 16];
T t0;
T t1;

void init() {
  t0 = {1, 1, 1, 1};
  t1 = {0, 0, 0, 0};

  dat[0] = Mono::unit();
  FOR(i, 16) {
    FOR(s, 1 << i) {
      T t = dat[s];
      dat[s] = Mono::op_R(t, t0);
      dat[s | 1 << i] = Mono::op_R(t, t1);
    }
  }
}

void apply(T& t, pi& p) {
  p.fi = p.fi + p.se * t.a + t.b;
  p.se = p.se * t.c + t.d;
}

u64 ADD[1 << 15];

void solve() {
  LL(N, Q);
  using BS = My_Bitset;
  STR(S);
  BS X = BS::from_string(S);

  auto solve = [&](BS x) -> void {
    ll n = len(x);
    pi ANS = {0, 0};
    while (n & 63) {
      if (x[n - 1]) {
        apply(t1, ANS);
      } else {
        apply(t0, ANS);
      }
      --n;
    }
    n /= 64;
    FOR_R(i, n) {
      u64 u = x.dat[i];
      // FOR_R(i, 64) {
      //   if (u >> i & 1) {
      //     apply(t1, ANS);
      //   } else {
      //     apply(t0, ANS);
      //   }
      // }
      // continue;
      u64 u0 = u & 65535;
      u64 u1 = (u >> 16) & 65535;
      u64 u2 = (u >> 32) & 65535;
      u64 u3 = (u >> 48) & 65535;
      apply(dat[u3], ANS);
      apply(dat[u2], ANS);
      apply(dat[u1], ANS);
      apply(dat[u0], ANS);
    }
    print(ANS.fi);
  };

  FOR(Q) {
    INT(t);
    if (t == 1) {
      INT(L, R);
      --L;
      while (L < R && (L % 64 != 0)) {
        X.flip(L++);
      }
      while (L < R && (R % 64 != 0)) {
        X.flip(--R);
      }
      L /= 64, R /= 64;
      ADD[L] ^= 1, ADD[R] ^= 1;
    }
    if (t == 2) {
      ll n = ceil<ll>(N, 64) + 1;
      FOR(i, n) {
        ADD[i + 1] ^= ADD[i];
        X.dat[i] ^= -ADD[i];
        ADD[i] = 0;
      }

      INT(l, a, b);
      --a, --b;
      BS x1 = X.slice(a, a + l);
      BS x2 = X.slice(b, b + l);
      BS x = x1 ^ x2;
      solve(x);
    }
  }
}

signed main() {
  init();
  solve();
}// END: main.cpp

```

```c++
template <typename T>
T floor(T a, T b)
{
    return a / b;
}

template <typename T>
T ceil(T x, T y)
{
    return floor(x + y - 1, y);
}

struct My_Bitset {
    int         N;
    vector<u64> dat;
    using T = My_Bitset;

    My_Bitset(int N = 0, int x = 0) : N(N)
    {
        assert(x == 0 || x == 1);
        u64 v = (x == 0 ? 0 : -1);
        dat.assign((N + 63) >> 6, v);
        if (N) dat.back() >>= (64 * len(dat) - N);
    }

    static T from_string(string &S)
    {
        int N = len(S);
        T   ANS(N);
        for (int i = 0; i < N; i++)
            ANS[i] = (S[i] == '1');
        return ANS;
    }

    int size()
    {
        return N;
    }

    // thanks to chatgpt!
    class Proxy {
    public:
        Proxy(vector<u64> &d, int i) : dat(d), index(i) {}

        operator bool() const
        {
            return (dat[index >> 6] >> (index & 63)) & 1;
        }

        Proxy &operator=(u64 value)
        {
            dat[index >> 6] &= ~(u64(1) << (index & 63));
            dat[index >> 6] |= (value & 1) << (index & 63);
            return *this;
        }

        void flip()
        {
            dat[index >> 6] ^= (u64(1) << (index & 63));  // XOR to flip the bit
        }

    private:
        vector<u64> &dat;
        int          index;
    };

    Proxy operator[](int i)
    {
        return Proxy(dat, i);
    }

    My_Bitset range(int L, int R)
    {
        assert(L <= R);
        My_Bitset p(R - L);
        int       rm = (R - L) & 63;
        for (i64 _ = 0; _ < rm; _++) {
            p[R - L - 1] = bool((*this)[R - 1]);
            --R;
        }
        int n  = (R - L) >> 6;
        int hi = L & 63;
        int lo = 64 - hi;
        int s  = L >> 6;
        if (hi == 0) {
            for (i64 i = 0; i < n; i++) {
                p.dat[i] ^= dat[s + i];
            }
        } else {
            for (i64 i = 0; i < n; i++) {
                p.dat[i] ^= (dat[s + i] >> hi) ^ (dat[s + i + 1] << lo);
            }
        }
        return p;
    }

    T &operator^=(const T &p)
    {
        assert(N == p.N);
        for (int i = 0; i < len(dat); i++)
            dat[i] ^= p.dat[i];
        return *this;
    }

    void flip(int i)
    {
        (*this)[i].flip();
    }

    T operator^(const T &p) const
    {
        return T(*this) ^= p;
    }
};

struct T {
    i64 hi, curans, through, lo;
};

struct Mono {
    using value_type = T;
    using X          = value_type;

    static X op(X L, X R)
    {
        // swap(L, R);
        // c是through
        // a是前缀
        // b是答案
        // d是后缀

        return T{L.hi + R.hi * L.through, L.curans + R.hi * L.lo + R.curans, R.through * L.through,
                 R.through * L.lo + R.lo};
    }

    static constexpr X unit()
    {
        return T{0, 0, 1, 0};
    }

    static constexpr bool commute = 0;
};

T dat[1 << 16];
T t0;
T t1;

void init()
{
    t0 = {1, 1, 1, 1};
    t1 = {0, 0, 0, 0};

    dat[0] = Mono::unit();
    for (int i = 0; i < 16; i++) {
        for (int s = 0; s < (1ll << i); s++) {
            T t             = dat[s];
            dat[s]          = Mono::op(t0, t);
            dat[s | 1 << i] = Mono::op(t1, t);
        }
    }
}

void apply(T &t, pair<i64, i64> &p)
{
    p.first  = p.first + p.second * t.hi + t.curans;
    p.second = p.second * t.through + t.lo;
}

signed main(signed argc, char **argv)
{
    init();
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
#ifdef BATCH
    freopen(argv[1], "r", stdin);
    freopen(argv[2], "w", stdout);
#endif
    int N, q;
    cin >> N >> q;
    string s;
    cin >> s;
    My_Bitset bitst(N);
    for (int i = 0; i < N; i++) {
        if (s[i] == '1') {
            bitst[i] = 1;
        }
    }
    auto work = [&](My_Bitset x) -> i64 {
        auto           start = Mono::unit();
        pair<i64, i64> ANS   = {0, 0};
        i64            n     = len(x);
        while (n & 63) {
            if (x[n - 1]) {
                apply(t1, ANS);
            } else {
                apply(t0, ANS);
            }
            --n;
        }
        n /= 64;
        for (int i = n - 1; i >= 0; i--) {
            u64 u = x.dat[i];
            // FOR_R(i, 64) {
            //   if (u >> i & 1) {
            //     apply(t1, ANS);
            //   } else {
            //     apply(t0, ANS);
            //   }
            // }
            // continue;
            u64 u0 = u & 65535;
            u64 u1 = (u >> 16) & 65535;
            u64 u2 = (u >> 32) & 65535;
            u64 u3 = (u >> 48) & 65535;
            // // apply(dat[u3], ANS);
            // start = Mono::op(start, dat[u3]);
            // start = Mono::op(start, dat[u2]);
            // start = Mono::op(start, dat[u1]);
            // start = Mono::op(start, dat[u0]);
            apply(dat[u3], ANS);
            apply(dat[u2], ANS);
            apply(dat[u1], ANS);
            apply(dat[u0], ANS);
        }
        return ANS.first;
    };
    vector<u64> block(1 << 15);
    for (int i = 0; i < q; i++) {
        int cmd;
        cin >> cmd;
        if (cmd == 1) {
            i64 l, r;
            cin >> l >> r;
            l--;
            while (l % 64 != 0 && l < r) {
                bitst.flip(l++);
            }
            while (r % 64 != 0 && l < r) {
                bitst.flip(--r);
            }
            block[r / 64] ^= 1;
            block[l / 64] ^= 1;
        } else {
            i64 n = ceil<i64>(N, 64);
            FOR(i, n)
            {
                block[i + 1] ^= block[i];
                bitst.dat[i] ^= -block[i];
                block[i] = 0;
            }
            i64 l, a, b;
            cin >> l >> a >> b;
            a--, b--;
            auto aa = bitst.range(a, a + l);
            auto bb = bitst.range(b, b + l);
            cout << work(aa ^ bb) << '\n';
        }
    }
    return 0;
}

// 0100
// 0010
//
// 0010
// 0100
//
//
// 1001001001
//
// 1110111001
//  23456
// 11011
// 11001
//

/* stuff you should look for
 * int overflow, array bounds
 * special cases (n=1?)
 * do smth instead of nothing and stay organized
 * WRITE STUFF DOWN
 * DON'T GET STUCK ON ONE APPROACH
 */
```