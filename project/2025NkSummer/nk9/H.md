---
title: Counter Streak
date: 2025-09-03
last_clear: 2025-09-03
tags:
  - tag/模拟
  - tag/dp
grasp:
difficulty: 1900
optional_difficulty: Medium
source: https://ac.nowcoder.com/acm/contest/108306/H
article:
---
# 题意
一个每日签到系统可以通过调整设备时区来维持连胜。  

给定一些在 UTC+8 的提交记录，每个提交记录都可以修改时区为 UTC-12 到 UTC+14 中的任意位置，  
问这样修改出来的最长连胜天数的最大值和最小值分别是多少。  

数据范围：$n \le 10^5$，时间戳在 2000 年至 2999 年之间。
## Input
- 第一行包含一个整数 $n$ $(1 \le n \le 10^5)$ — 提交记录的数量。  
- 接下来的 $n$ 行，每行包含一个时间戳，格式为 `YYYY-MM-DD HH:mm:SS`（UTC+8），保证按时间顺序排列。  
保证所有日期均在 2000-01-01 至 2999-12-31 之间。
## Output
输出两个用空格分隔的整数：最长连胜天数的最大可能值和最小可能值。
# Solution
## my
直接离散化，然后dp
```cpp showLineNumbers title="lace"
long long funcB(int year, int month, int day, int h, int m, int s)
{
    tm t      = {};
    t.tm_year = year - 1900;  // tm_year = 年份-1900
    t.tm_mon  = month - 1;    // tm_mon = 0~11
    t.tm_mday = day;
    t.tm_hour = h;
    t.tm_min  = m;
    t.tm_sec  = s;
    time_t tt = timegm(&t);  // 转成 time_t（秒）
    if (tt == -1) return -1;
    tt -= 20 * 3600;
    return tt / 86400;
}

long long funcA(int year, int month, int day, int h, int m, int s)
{
    tm t      = {};
    t.tm_year = year - 1900;  // tm_year = 年份-1900
    t.tm_mon  = month - 1;    // tm_mon = 0~11
    t.tm_mday = day;
    t.tm_hour = h;
    t.tm_min  = m;
    t.tm_sec  = s;
    time_t tt = timegm(&t);  // 转成 time_t（秒）
    if (tt == -1) return -1;
    tt += 6 * 3600;
    return tt / 86400;
}

void solve()
{
    int n;
    scanf("%d", &n);
    vector<i64> days;
    for (int i = 0; i < n; i++) {
        int yy, mm, dd, h, m, s;
        // 2020-02-29 09:10:23
        scanf("%d-%d-%d %d:%d:%d", &yy, &mm, &dd, &h, &m, &s);
        i64 a = funcA(yy, mm, dd, h, m, s);
        i64 b = funcB(yy, mm, dd, h, m, s);
        for (i64 j = a; j >= b; j--) {
            days.pb(j);
        }
    }
    vector<i64> lisanhua;
    for (i64 i : days) {
        lisanhua.pb(i - 1), lisanhua.pb(i), lisanhua.pb(i + 1);
    }
    sor(lisanhua);
    lisanhua.erase(unique(all(lisanhua)), lisanhua.end());
    int num = lisanhua.size();
    for (i64& i : days) {
        i = lower_bound(all(lisanhua), i) - lisanhua.begin();
    }
    vector<i64> dp(num);
    for (i64 i : days) {
        dp[i] = max(dp[i - 1] + 1, dp[i]);
    }
    i64 ans = 0;
    ans     = *max_element(all(dp));
    printf("%lld 1", ans);
}
```
