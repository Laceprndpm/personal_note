---
title: I, Box
date: 2025-09-01
last_clear: 2025-09-01
tags:
  - tag/模拟
  - tag/大模拟
grasp: 2
difficulty: 2000
source: https://ac.nowcoder.com/acm/contest/108301/I
article:
---
# 题意
给定 $n \times m$ 的推箱子地图，但是规则变更为不需要人去推，而是箱子可
以自己移动（同样不能移动到障碍物或者其他箱子上）。要求判断是否
存在解，如果存在解要求构造出一组总移动次数不超过 $10^5$ 的解。
## Input
- 第一行包含三个整数 $n, m, k$ $(1 \le n, m \le 50)$，分别表示地图的高度、宽度和箱子的数量。  
- 接下来有 $n$ 行，每行包含一个长度为 $m$ 的字符串，表示地图。每个字符属于集合 {`#`， `.`，`@`，`*`，`!`}，其含义如下：  
  - `#` 表示位置 $(i, j)$ 有一堵墙。  
  - `.` 表示位置 $(i, j)$ 是空的。  
  - `@` 表示位置 $(i, j)$ 有一个箱子。  
  - `*` 表示位置 $(i, j)$ 有一个目标位置。  
  - `!` 表示位置 $(i, j)$ 同时有一个箱子和一个目标位置。  
约束条件：
- 地图被墙包围；  
- 箱子的数量等于目标数量；  
- 地图中最多有 50 个箱子。
## Output
- 如果不存在有效的序列将所有箱子移动到目标位置，输出 `-1`。  
- 否则，第一行输出一个整数 $L$ $(0 \le L \le 10^5)$，表示移动序列的长度。  
- 接下来的 $L$ 行，每行包含两个整数和一个字符：  
  $$
  x\ y\ c \quad (1 \le x \le n, 1 \le y \le m, \ c \in \{L, R, U, D\})
  $$  
  这表示将当前位置为 $(x, y)$ 的箱子向左、右、下或上移动一步，即移动到 $(x, y-1)$、$(x, y+1)$、$(x-1, y)$ 或 $(x+1, y)$。  

注意事项：

- 不应将任何箱子移动到墙或其他箱子所在的位置。  
- 如果存在多个有效的移动序列，输出其中任意一个即可被视为正确。  
- 可以证明，在给定约束下，如果存在任何有效序列，则一定存在长度不超过 $10^5$ 的序列。
# Solution
## std
考虑整个地图被障碍物分成的多个连通区域。如果某个连通区域中箱子
数量不等于终点数量，那么显然无解。
否则原问题一定存在解。每次考虑通过选取一条不经过障碍物的最短路
径，将一个不在终点上的箱子推到一个上面没有箱子的终点上。如果这
条路径上依次存在其他箱子 $a_1, a_2, . . . , a_l$, 只需要先将 $a_l$ 推到终点，将
$a_l−1$推到 $a_l$原先所在位置... 并以此类推即可。
由于每次选取的是最短路，可以证明这样的操作次数一定不超过 $10^5$。
总的时间复杂度为 $O(50nm)$。
### 补充
一定要提前规划好代码的具体实现，在规划过程中就要注意，哪些条件是等价的，哪些是看似等价的！这里不小心把 `if (ax == x && ay == y)` 写成了 `if(board[ax] == '*')` 导致浪费大量时间！
```cpp showLineNumbers title="mycode"
void solve()
{
    constexpr int dx[] = {0, 0, -1, +1};
    constexpr int dy[] = {-1, +1, 0, 0};
    int           n, m;
    cin >> n >> m;
    vector<string> board(n);
    for (int i = 0; i < n; i++) {
        cin >> board[i];
    }
    bool is_solveable = [&]() -> bool {
        vector<vector<int>> visited(n, vector<int>(m));

        auto isValid = [&n, &m, &board](int x, int y) {
            return (0 <= x && x < n) && (0 <= y && y < m) && board[x][y] != '#';
        };
        auto dfs = [&](auto&& dfs, int x, int y) -> int {
            if (!isValid(x, y)) return 0;
            if (visited[x][y]) return 0;
            visited[x][y] = 1;
            int val       = 0;
            switch (board[x][y]) {
                case '@': val += 1; break;
                case '*': val -= 1; break;
            }
            for (int i = 0; i < 4; i++) {
                int nx = x + dx[i];
                int ny = y + dy[i];
                val += dfs(dfs, nx, ny);
            }
            return val;
        };
        int ok = 1;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (dfs(dfs, i, j) != 0) {
                    ok = 0;
                }
            }
        }
        return ok;
    }();
    if (!is_solveable) {
        cout << -1 << '\n';
        return;
    }
    vector<array<int, 3>> ans_string;

    auto bfs = [&](const int x, const int y) -> void {
        auto isValid = [&n, &m, &board](int x, int y) {
            return (0 <= x && x < n) && (0 <= y && y < m) && board[x][y] != '#';
        };
        vector<vector<int>>         father(n, vector<int>(m, {-1}));
        deque<tuple<int, int, int>> que;
        que.push_back({x, y, 4});
        int ax, ay;
        while (true) {
            assert(!que.empty());
            auto [cx, cy, sta] = que.front();
            que.pop_front();
            if (!isValid(cx, cy)) continue;
            if (father[cx][cy] != -1) continue;
            father[cx][cy] = sta;
            if (board[cx][cy] == '@') {
                ax = cx;
                ay = cy;
                break;
            }
            for (int i = 0; i < 4; i++) {
                int nx = cx + dx[i];
                int ny = cy + dy[i];
                que.push_back({nx, ny, i});
            }
        }
        int destx = ax, desty = ay;

        vector<vector<array<int, 3>>> tmp_ans;
        vector<array<int, 3>>         cur_string;
        while (true) {
            if (board[ax][ay] == '!') {
                tmp_ans.push_back(cur_string);
                cur_string.clear();
            }
            if (ax == x && ay == y) {
                tmp_ans.push_back(cur_string);
                break;
            }
            assert(!(board[ax][ay] == '@' && ax != destx && ay != desty));
            assert(father[ax][ay] != 4);
            int sta = father[ax][ay] ^ 1;
            cur_string.push_back({ax, ay, sta});
            ax = ax + dx[sta];
            ay = ay + dy[sta];
        }
        reverse(all(tmp_ans));
        for (auto& it : tmp_ans) {
            ans_string.insert(ans_string.end(), it.begin(), it.end());
        }
        board[destx][desty] = '.';
        board[x][y]         = '!';
    };
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            if (board[i][j] == '*') {
                bfs(i, j);
            }
        }
    }
    cout << ans_string.size() << '\n';
    constexpr char cx[] = {'L', 'R', 'U', 'D'};
    for (auto& it : ans_string) {
        cout << it[0] + 1 << ' ' << it[1] + 1 << ' ' << cx[it[2]];
        cout << '\n';
    }
}
```
