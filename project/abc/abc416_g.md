---
title: G - Concat (1st)
date: 2025-07-26
last_clear: 2025-07-26
tags:
  - tag/贪心
  - tag/bfs
  - tag/thinking/reverse
  - amazing
grasp: 2
difficulty: 1800
source: https://atcoder.jp/contests/abc416/tasks/abc416_g
article:
---
# My_Solution
```text
    // 最大的问题，即不同长度间，性价比无法比较
    // abc
    // abcaa
    // 后者明显优于前者，而这又取决于其他字符串是否有aaa。。。
    // 哦，长度最多为10,感觉lcm() = 2520 + 某种dp，a + b 对比 c + d
    // 无穷背包？
    //
    // 贪心：
    // 先按照顺序排序，选择最短的哪个，检查，直到字典序>的位置
    // 此时，就获得了10个长度不同开头的，对于每个长度，都可以再来一次？把所有上面的字符串前缀加入后
    // 或者也许精确地说，我们只需要每个长度中最小的？
    //
```
总之，就是 `bfs` ，但是状态更新要想的细致一些
```c++
// update:
for (int i = 1; i <= 10; i++) {
    if (candi[i].empty()) continue;
    int update = 1;  // 默认更新
    for (int j = 1; j <= i; j++) {
        if (min_c[cur_len + j] > candi[i][j]) {  // 强更新
            min_c[cur_len + j] = candi[i][j];
            update             = 2;
            break;
        } else if (min_c[cur_len + j] < candi[i][j]) {  // 无更新
            update = 0;
            break;
        }
    }
    if (update == 2) {  // 强更新
        int j;
        for (j = 1; j <= i; j++) {
            min_c[cur_len + j] = candi[i][j];
        }
        for (; j <= 10; j++) {
            min_c[cur_len + j] = 'z' + 1;
        }
    }
    if (update) {  // 同时放堆里面去
        string new_string =
            string(s.begin() + i, s.begin() + 11) + string(candi[i].begin() + 1, candi[i].end());
        assert(new_string.size() == 11);
        pq.push({cur_len + i, new_string, cur.used + 1});
    }
}
```
# Amazing_Solution
```c++
    for (int i = 0; i < K; i++) {
        if (i <= 10) {
            sort(S.begin(), S.end(), [&](string s, string t) {
                return s + ans < t + ans;
            });
            ans = S[0] + ans;
        } else {
            string ans2;
            for (int j = i; j < K; j++) {
                ans2 += S[0];
            }
            ans2 += ans;
            ans = ans2;
            break;
        }
    }
```
假设从后往前构造
$a_1 + suffix$ 为最优序列
若 $a_2 + suffix \le a_1 + suffix$
则选 $a_2+suffix$ 并不会使结果更劣，因为其新增同一前缀时，绝不会更劣
## 为什么10轮后就可以直接贪心？
考虑为什么会发生$cba > c$ 的情况，是不是因为我们不知道后面放的是 $aaa$ 还是 $ccc$ ？但当循环10次后，长度必然 > 10，则 $cba + suffix < c + suffix$ 始终不变
**应该不要想到==补全==就想到lcm，有时候有更朴素的性质**
```c++
void solve()
{
    int n, k;
    cin >> n >> k;
    vector<string> s_vec(n);
    string         ans;
    for (int i = 0; i < n; i++) {
        cin >> s_vec[i];
    }
    int i = 0;
    for (; i < k && sz(ans) < 10; i++) {
        sort(all(s_vec), [&](string a, string b) {
            return a + ans < b + ans;
        });
        ans = s_vec[0] + ans;
    }
    for (; i < k; i++) {
        cout << s_vec[0];
    }
    cout << ans << '\n';
}
```
