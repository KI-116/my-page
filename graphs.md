# 图论

## 岛屿问题


### 题目描述

四面都是水。
```
input:
grid = [
  ['1','1','1','1','0'],
  ['1','1','0','1','0'],
  ['1','1','0','0','0'],
  ['0','0','0','0','0']
]

output:1
```
### 深度搜索 (DFS)

1. 确认递归函数参数 check the arguments of the recursive function

`void dfs(const vector<vector<char>>& grid, int x, int y)`

如果需要保存搜索路径，可以增加一个 `visited` 数组，即 `vector<vector<bool>> visited`

2. 确认递归终止条件 determine the termination condition of recursion

终止条件达成，则返回。
或 无显性终止条件，不符合条件即不继续递归。

3. 确认单层递归逻辑 the logic of a single layer of recursion

``` cpp
for (选择：本节点所连接的其他节点) {
    处理节点;
    dfs(图，选择的节点); // 递归
    回溯，撤销处理结果
}
```

#### 思路分析

当遇到一个陆地 '1' 时，我们可以通过深度优先搜索（DFS）将与该陆地相连的所有陆地都标记为已访问（例如，将它们改为 '0'）。这样，每次我们发现一个新的陆地 '1' 时，就意味着我们找到了一个新的岛屿。


#### 代码实现
``` cpp
class Solution {
private:
  void dfs(vector<vector<char>>& grid, int x, int y) {
    int m = grid.size();
    int n = grid[0].size();
    grid[x][y] = '0'; // 标记为已访问
    // 上下左右四个方向
    if (x - 1 >= 0 && grid[x - 1][y] == '1') {
      dfs(grid, x - 1, y);
    }
    if (x + 1 < m && grid[x + 1][y] == '1') {
      dfs(grid, x + 1, y);
    }
    if (y - 1 >= 0 && grid[x][y - 1] == '1') {
      dfs(grid, x, y - 1);
    }
    if (y + 1 < n && grid[x][y + 1] == '1') {
      dfs(grid, x, y + 1);
    }
  }
public:
  int numIslands(vector<vector<char>>& grid) {
    int m = grid.size();
    if (m == 0) return 0;
    int n = grid[0].size();   
    int num_islands = 0;
    for (int x = 0; x < m; ++x) {
      for (int y = 0; y < n; ++y) {
        if (grid[x][y] == '1'){
          ++num_islands;
          dfs(grid, x, y);
        }
      }
    }
    return num_islands;
  }
    
};
    


```
### 广度搜索 (BFS)
一般来说，BFS 适用于寻找最短路径的问题。

#### 保存状态
使用队列 `queue<pair<int, int>> q` 来保存当前层的节点：  
使用栈 `stack<pair<int, int>> s` 来保存当前层的节点（不常用）。
#### 思路分析


## 腐烂的橘子
### 题目描述
在给定的网格中，每个单元格可以有以下三个值之一：
- 值 0 代表空单元格；
- 值 1 代表新鲜橘子；
- 值 2 代表腐烂的橘子。

每分钟，任何与腐烂的橘子（在 4 个正方向上相邻）相邻的新鲜橘子都会腐烂。
返回直到单元格中没有新鲜橘子为止所必须经过的最小分钟数。如果不可能，返回 -1。