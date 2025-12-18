极其快速的模运算，原理未知
```cpp
#include <cstdint>
#include <immintrin.h>

using i32 = std::int32_t;
using u32 = std::uint32_t;
using u64 = std::uint64_t;

struct mont {
    u32 n, n2, r, r2, ir, _1;
    constexpr mont(u32 n) : n(n), n2(n * 2), r((u64(1) << 32) % n), r2(u64(r) * r % n), ir(1) {
        for (u32 i = 0; i < 5; i++) ir *= 2 - ir * n;
        ir = -ir, _1 = trans(1);
    }
    constexpr u32 reduce(u64 x) { return (x + u64(u32(x) * ir) * n) >> 32; }
    constexpr u32 trans(u32 x) { return reduce(u64(x) * r2); }
    constexpr u32 get(u32 x) {
        u32 v0 = reduce(x), v1 = v0 - n;
        return i32(v1) < 0 ? v0 : v1;
    }
    constexpr u32 neg(u32 x) { return x ? n2 - x : 0; }
    constexpr u32 add(u32 x, u32 y) {
        u32 v0 = x + y, v1 = v0 - n2;
        return i32(v1) < 0 ? v0 : v1;
    }
    constexpr u32 sub(u32 x, u32 y) {
        u32 v0 = x - y, v1 = v0 + n2;
        return i32(v0) < 0 ? v1 : v0;
    }
    constexpr u32 mul(u32 x, u32 y) { return reduce(u64(x) * y); }
    constexpr u32 qpow(u32 x, u64 n, u32 r) {
        for (; n; n >>= 1, x = mul(x, x))
            if (n & 1) r = mul(r, x);
        return r;
    }
    constexpr u32 qpow(u32 x, u64 n) { return qpow(x, n, _1); }
    constexpr u32 inv(u32 x) { return qpow(x, n - 2); }
    constexpr u32 div(u32 x, u32 y) { return qpow(y, n - 2, x); }
};

#include <cstdint>
#include <cstring>
#include <sys/mman.h>
#include <sys/stat.h>
namespace io {
struct stat s;
char *p;
int n[0x10000];
__attribute__((constructor)) inline void init() {
    fstat(0, &s);
    p = (char *)mmap(nullptr, s.st_size, PROT_READ, MAP_PRIVATE, 0, 0);
    memset(n, -1, sizeof(n));
    for (int i = '0', x = 0; i <= '9'; i++)
        for (int j = '0'; j <= '9'; j++) n[i ^ (j << 8)] = x++;
}
inline void read(auto &x) {
    while (*p < '0') ++p;
    x = *p++ ^ '0';
    while (~n[*(uint16_t *)p]) x = x * 100 + n[*(uint16_t *)p], p += 2;
    if (*p >= '0') x = x * 10 + (*p++ ^ '0');
}
inline void read_s(auto &x) {
    bool f = false;
    while (*p < '0') f = *p++ == '-';
    x = *p++ ^ '0';
    while (~n[*(uint16_t *)p]) x = x * 100 + n[*(uint16_t *)p], p += 2;
    if (*p >= '0') x = x * 10 + (*p++ ^ '0');
    if (f) x = -x;
}
inline void read(auto &&...args) { (read(args), ...); }
inline void read_s(auto &&...args) { (read_s(args), ...); }
} // namespace io

#include <cstdio>
const int N = 5'000'001;
u32 a[N], s[N];
int main() {
    int n;
    u32 p, k;
    io::read(n, p, k);
    mont mt(p);
    k = mt.trans(k);
    u32 ik = mt.inv(k);
    u32 kn = mt._1;
    s[0] = mt._1;
    for (int i = 1; i <= n; i++) {
        io::read(a[i]);
        a[i] = mt.trans(a[i]);
        kn = mt.mul(kn, ik);
        a[i] = mt.mul(a[i], kn);
        s[i] = mt.mul(s[i - 1], a[i]);
    }
    u32 tn = mt.inv(s[n]);
    u32 ans = mt.mul(tn, s[n - 1]);
    for (int i = n - 1; i >= 1; i--) {
        tn = mt.mul(tn, a[i + 1]);
        u32 ia = mt.mul(s[i - 1], tn);
        ans = mt.add(ans, ia);
    }
    printf("%u", mt.get(ans));
}
```
