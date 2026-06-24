# 目录
- [滑动窗口](#滑动窗口)
- [回溯算法](#回溯算法)
- [动态规划](#动态规划)
- [滑动窗口](#滑动窗口)
- [岛屿问题](#岛屿问题)
- [栈与队列：前 K 个高频元素](#栈与队列：前-K-个高频元素)
- [二叉树](#二叉树)
    - [二叉树递归：后序](#二叉树递归后序)
    - [二叉树层序](#二叉树层序)

# 滑动窗口

``` python
// 滑动窗口模板
int left = 0, right = 0;
while (right < s.size()) {
    // 扩大窗口
    window.add(s[right]);
    right++;

    while (窗口满足条件) {
        // 更新结果
        ...

        // 缩小窗口
        window.remove(s[left]);
        left++;
    }
}
```

# 回溯算法
场景：子集，组合，全排列，分割

```
void backtracking(参数) {
    if (终止条件) {
        存放结果(push_back(path)/append(path[:]));
        return;
    }
    for (选择：本层集合中的元素) {
        处理节点(path.push_back(i)/path.append(i));
        backtracking(路径，选择列表);
        撤销处理结果(path.pop_back()/path.pop());
    }
}
```

时间复杂度：O(n!)或者O（2^n），空间复杂度：O(k)（递归深度）,一般是O(n)（递归深度是n,每层空间常数级别）


# 动态规划

```
// 1. 定义状态：dp[i] = ...  // dp 数组的含义
// 2. 定义状态转移方程：dp[i] = ...  // 根据之前的状态转移到当前状态
// 3. 初始化：dp[0] = ...  // 根据状态定义初始化
// 4. 计算顺序：for (int i = 1; i < n; i++) { ... }  // 根据状态转移方程计算 dp 数组
```

# 滑动窗口
**极简模板：**
```python
l=0; 
for r in range(n): 
    更新窗口数据； 
    while 窗口不满足条件: 
        更新数据并 l++, 直到满足条件 
    更新答案
```
``` python
def lengthOfLongestSubstring(s: str) -> int:
    window = set()
    left = 0
    res = 0
    
    for right in range(len(s)):
        # 只要新加入的字符在窗口里存在，就一直收缩左边界，直到不重复
        while s[right] in window:
            window.remove(s[left])
            left += 1
            
        # 此时窗口内一定没有重复字符了，加入新字符
        window.add(s[right])
        # 更新最大长度（在 while 外部更新，因为此时窗口合法且最大）
        res = max(res, right - left + 1)
        
    return res
```

# 岛屿问题
``` python
from collections import deque
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

# 栈与队列：前 K 个高频元素
``` python
#时间复杂度：O(nlogk)
#空间复杂度：O(n)
import heapq
class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        #要统计元素出现频率
        map_ = {} #nums[i]:对应出现的次数
        for i in range(len(nums)):
            map_[nums[i]] = map_.get(nums[i], 0) + 1
        
        #对频率排序
        #定义一个小顶堆，大小为k
        pri_que = [] #小顶堆
        
        #用固定大小为k的小顶堆，扫描所有频率的数值
        for key, freq in map_.items():
            heapq.heappush(pri_que, (freq, key))
            if len(pri_que) > k: #如果堆的大小大于了K，则队列弹出，保证堆的大小一直为k
                heapq.heappop(pri_que)
        
        #找出前K个高频元素，因为小顶堆先弹出的是最小的，所以倒序来输出到数组
        result = [0] * k
        for i in range(k-1, -1, -1):
            result[i] = heapq.heappop(pri_que)[1]
        return result
```

# 二叉树
## 二叉树递归：后序
``` python
class Solution:
    def lowestCommonAncestor(self, root, p, q):
        if root == q or root == p or root is None:
            return root

        left = self.lowestCommonAncestor(root.left, p, q)
        right = self.lowestCommonAncestor(root.right, p, q)

        if left is not None and right is not None:
            return root

        if left is None and right is not None:
            return right
        elif left is not None and right is None:
            return left
        else: 
            return None
```

## 二叉树层序
``` python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if not root:
            return []
        queue = collections.deque([root])
        result = []
        while queue:
            level = []
            for _ in range(len(queue)):
                cur = queue.popleft()
                level.append(cur.val)
                if cur.left:
                    queue.append(cur.left)
                if cur.right:
                    queue.append(cur.right)
            result.append(level)
        return result
```

