# 目录
- [双指针](#双指针)
    - [two ends](#two-ends)
    - [快慢指针](#快慢指针)
    - [滑动窗口](#滑动窗口)
- [回溯算法](#回溯算法)
    - [组合/子集](#组合子集)
    - [全排列](#全排列)
- [堆排序](#堆排序)
- [动态规划](#动态规划)
- [拓扑排序](#拓扑排序)
- [岛屿问题](#岛屿问题)
- [单调栈](#单调栈)
- [二叉树](#二叉树)
    - [二叉树递归：后序](#二叉树递归后序)
    - [二叉树层序](#二叉树层序)

# 双指针
## two ends

- 有序数组
- 回文串
- 容器盛水
- 两数之和 II

``` python
left, right = 0, len(nums) - 1

while left < right:
    if 满足条件:
        return ...

    elif 需要左指针移动:
        left += 1

    else:
        right -= 1
```


## 快慢指针

- 链表
- 数组原地删除
- 去重
- Floyd 判环

``` python
slow = 0

for fast in range(len(nums)):
    if 保留 nums[fast]:
        nums[slow] = nums[fast]
        slow += 1

return slow
```

## 滑动窗口

``` python
left = 0

for right in range(len(nums)):
    add(nums[right])

    while 窗口非法:
        remove(nums[left])
        left += 1

    更新答案()
```


# 回溯算法
时间复杂度：O(n!)或者O（2^n），空间复杂度：O(k)（递归深度）,一般是O(n)（递归深度是n,每层空间常数级别）
``` python
res = []
path = []
def backtracking(...):
    if 满足终止条件:
        res.append(path[:])
        return
    #枚举本层集合中的所有元素
    for i in range(...):
        if 不合法:#剪枝（可选）
            continue
        path.append(i)
        backtracking(...)
        path.pop()

backtracking(...)
return res
```

## 组合/子集
``` python
def backtracking(start):

    if 结束条件:
        res.append(path[:])
        return

    for i in range(start, len(nums)):
        path.append(nums[i])
        backtracking(i + 1)
        path.pop()
```

## 全排列
``` python  
res = []
path = []
used = set()

def backtracking():

    if len(path) == len(nums):
        res.append(path[:])
        return

    for i in range(len(nums)):
        if i in used:
            continue

        used.add(i)
        path.append(nums[i])

        backtracking()

        path.pop()
        used.remove(i)
```

# 堆排序
``` python
import heapq
heap = []
for i in range(n):
    heapq.heappush(heap, nums[i])  # O(logn)
    if len(heap) > k:
        heapq.heappop(heap)  # O(logn)
```
# 动态规划

1. 一维
```
// 1. 定义状态：dp[i] = ...  // dp 数组的含义
// 2. 定义状态转移方程：dp[i] = ...  
// 3. 初始化：dp = [0]*len(nums)
// 4. 计算顺序：for (int i = 1; i < n; i++) { ... }  // 根据状态转移方程计算 dp 数组
```

2. 二维
```
// 1. 定义状态：dp[i][j] = ...  // dp 数组的含义
// 2. 定义状态转移方程：dp[i][j] = ...
// 3. 初始化：dp = [[False]*len(s) for _ in range(len(s))]
// 4. 确定遍历顺序:如果是逆序，for i in range(len(s)-1, -1, -1): for j in range(i+1, len(s)): ...;如果j也逆序，for i in range(len(s)-1, -1, -1): for j in range(len(s)-1, i, -1): ...
```


# 拓扑排序


1. 找到入度为0 的出发节点，加入结果集
2. 将该节点从图中移除
循环以上两步，直到所有节点都在图中被移除了。结果集的顺序，就是我们想要的拓扑排序顺序。
如果发现结果集元素个数**不等于**图中节点个数，就可以认定图中一定有 有向环！

# 岛屿问题
``` python
m, n = len(grid), len(grid[0])
dirs = [(1,0), (-1,0), (0,1), (0,-1)]

def dfs(i, j):
    # 1. 越界
    if i < 0 or i >= m or j < 0 or j >= n:
        return

    # 2. 非法位置（水/已访问）
    if grid[i][j] != LAND:
        return

    # 3. 标记访问
    grid[i][j] = VISITED

    # 4. 继续搜索四个方向
    for dx, dy in dirs:
        dfs(i + dx, j + dy)

for i in range(m):
    for j in range(n):
        if grid[i][j] == LAND:
            # 根据题意统计答案
            dfs(i, j)
```

# 单调栈    


``` python
stack = []

for i in range(len(nums)):
    # 当前元素和栈顶比较，维持栈的单调性
    # nums[i] 就是 idx 右边第一个更大元素
    while stack and nums[stack[-1]] < nums[i]:
    # while stack and nums[stack[-1]] > nums[i]: 右边第一个更小元素
    # while stack and nums[stack[-1]] <= nums[i]: 左边第一个更大元素
    # while stack and nums[stack[-1]] >= nums[i]: 左边第一个更小元素
        idx = stack.pop()

    stack.append(i)
```

- 下一个更大元素（Next Greater Element）
- 下一个更小元素
- 上一个更大元素
- 上一个更小元素
- 第一个满足条件的元素
- 最近更高/更低
- 每个元素作为最大值/最小值的贡献
- 柱状图、矩形面积

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

