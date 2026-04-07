# 图论

[return to Index]:<https://ki-116.github.io/my-page/>

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
1. 初始化队列，将起始节点入队。
``` cpp
queue<pair<int, int>> q;
q.push({start_x, start_y});
```
2. 进入循环，直到队列为空。
``` cpp
while (!q.empty()) {
    // 处理当前节点:从队列中取出节点
    auto [x, y] = q.front();
    q.pop();
    // 处理节点逻辑：上下左右，等等
    for (选择：本节点所连接的其他节点) {
        处理节点;
        if (节点未被访问)
          q.push(选择的节点); // 将相邻节点入队
    }
}
```

#### 思路分析
BFS 在本题中不需要拿出节点进行处理，而是直接将节点的相邻节点入队，并将相邻节点标记为已访问。
如果遇到一个陆地 '1'，就将其相邻的陆地全部入队，并标记为已访问，直到队列为空，表示该岛屿的所有陆地都被访问过了。

#### 代码实现
``` cpp
class Solution {
public:
  int numIslands(vector<vector<char>>& grid) {
    int m = grid.size();
    if(!m) return 0;
    int n = grid[0].size();
    int num_islands = 0;
    for(int x = 0; x < m; ++x) {
      for(int y = 0; y < n; ++y) {
        if(grid[x][y] == '1') {
          ++num_islands;
          grid[x][y] = '0'; // 标记为已访问
          queue<pair<int, int>> q;
          q.push({x, y});
          while(!q.empty()){
            auto [curr_x, curr_y] = q.front();
            q.pop();
            // 上下左右四个方向
            if (curr_x - 1 >= 0 && grid[curr_x - 1][curr_y] == '1') {
              grid[curr_x - 1][curr_y] = '0';
              q.push({curr_x - 1, curr_y});
            }
            if (curr_x + 1 < m && grid[curr_x + 1][curr_y] == '1') {
              grid[curr_x + 1][curr_y] = '0';
              q.push({curr_x + 1, curr_y});
            }
            if (curr_y - 1 >= 0 && grid[curr_x][curr_y - 1] == '1') {
              grid[curr_x][curr_y - 1] = '0';
              q.push({curr_x, curr_y - 1});
            }
            if (curr_y + 1 < n && grid[curr_x][curr_y + 1] == '1') {
              grid[curr_x][curr_y + 1] = '0';
              q.push({curr_x, curr_y + 1});
            }
          }
        }
      }
    }
    return num_islands;
  }
};
```

## 腐烂的橘子
### 题目描述
在给定的网格中，每个单元格可以有以下三个值之一：
- 值 0 代表空单元格；
- 值 1 代表新鲜橘子；
- 值 2 代表腐烂的橘子。

每分钟，任何与腐烂的橘子（在 4 个正方向上相邻）相邻的新鲜橘子都会腐烂。
返回直到单元格中没有新鲜橘子为止所必须经过的最小分钟数。如果不可能，返回 -1。


### 思路分析
在这里使用BFS，因为如题目描述，这是一个每分钟逐层扩展的情况。

1. 初始化队列，将所有腐烂的橘子入队，并统计新鲜橘子的数量。
2. 进入循环，直到队列为空。
3. 对于每一层（每分钟），处理当前队列中的所有腐烂橘子：
   - 对于每个腐烂橘子，检查其四个方向的相邻单元格。
   - 如果相邻单元格中有新鲜橘子，将其腐烂（标记为2），并将其入队，同时减少新鲜橘子的计数。
4. 如果在某一层中有新鲜橘子被腐烂，分钟数加1。
5. 循环结束后，检查是否还有新鲜橘子剩余。如果有，返回 -1；否则，返回分钟数。

> 腐烂橘子同时影响相邻新鲜橘子，初始化时需要将所有腐烂橘子入队。
> 每分钟处理一层队列，确保同时腐烂的效果。使用`bool rotten` 标记当前分钟是否有新鲜橘子被腐烂，以决定是否增加分钟数。（和cnt计算岛屿大小的区别：岛屿大小需要每次都计算，而分钟数只在有变化时增加）  
> 为什么不需要使用`visited`数组？因为可以直接修改原始网格。


### 代码实现
``` cpp
class Solution {
public:
    int orangesRotting(vector<vector<int>>& grid) {
        int min = 0, fresh = 0;
        queue<pair<int, int>> q;
        for(int i = 0; i < grid.size(); i++) {
            for(int j = 0; j < grid[0].size(); j++)
                if(grid[i][j] == 1) fresh++;
                else if(grid[i][j] == 2) q.push({i, j});
        }
        vector<pair<int, int>> dirs = { {-1, 0}, {1, 0}, {0, -1}, {0, 1} };
        while(!q.empty()) {
            int n = q.size();
            bool rotten = false;
            for(int i = 0; i < n; i++) {
                auto x = q.front();
                q.pop();
                for(auto cur: dirs) {
                    int i = x.first + cur.first;
                    int j = x.second + cur.second;
                    if(i >= 0 && i < grid.size() && j >= 0 && j < grid[0].size() && grid[i][j] == 1) {
                        grid[i][j] = 2;
                        q.push({i, j});
                        fresh--;
                        rotten = true;
                    }
                }
            }
            if(rotten) min++;
        } 
        return fresh ? -1 : min;
    }
};

```
## 课程表
### 题目描述
你这个学期必须选修 numCourses 门课程，记为 0 到 numCourses- 1 。在选修某些课程之前需要一些先修课程。  
例如，要想学习课程 0 ，你需要先完成课程 1 ，我们用一个匹配先修课程的数组 prerequisites 来表示它们之间的先修关系，其中 prerequisites[i] = [ai, bi] ，表示如果要学习课程 ai 则 必须 先学习课程 bi 。  
请你判断是否可能完成所有课程的学习？如果可以，返回 true；否则返回 false 。