# data structure(python)

速查：
[List](#1-list动态数组) | [String](#2-string字符串) | [Stack](#3-stack栈) | [Queue](#4-queue队列) | [Deque](#5-deque双端队列) | [Set](#6-set哈希集合) | [Dict](#7-dict哈希表) | [Counter](#8-counter计数器) | [defaultdict](#9-defaultdict) | [Heap](#10-heap优先队列) | [OrderedDict](#11-ordereddict很少) | [TreeNode](#12-treenode二叉树) | [ListNode](#13-listnode链表) | [Graph](#14-graph图) | [Trie](#15-trie前缀树) | [Union Find](#16-union-find并查集)

# 1. List（动态数组）

最常用的数据结构。

```python
nums = [1, 2, 3]
```

## 常用操作

```python
nums.append(4)      # 尾插
nums.pop()          # 尾删

nums.insert(0, 10)  # 指定位置插入
nums.remove(2)      # 删除值2

nums.reverse()      # 反转
nums.sort()         # 排序

nums[0]
nums[-1]

len(nums)
```

## 复杂度

| 操作            | 时间   |
| ------------- | ---- |
| append        | O(1) |
| pop()         | O(1) |
| 随机访问          | O(1) |
| insert/remove | O(n) |

---

# 2. String（字符串）

```python
s = "leetcode"
```

## 常用操作

```python
len(s)

s[0]

s[::-1]

s.find("abc")

s.replace("a", "b")

s.split(',')

",".join(arr)
```

字符串不可修改：

```python
s[0] = 'a'  # Error
```

修改时：

```python
arr = list(s)

arr[0] = 'a'

s = "".join(arr)
```

---

# 3. Stack（栈）

LeetCode 中直接用 list。

```python
stack = []
```

## 操作

```python
stack.append(x)
stack.pop()
stack[-1] # 查看栈顶元素
# 判断栈是否为空
if not stack:
    print("Stack is empty")
```

## 模板

```python
stack = []

for c in s:

    if 条件:
        stack.pop()

    stack.append(c)
```

## 高频题

* 20 有效括号
* 394 字符串解码
* 739 每日温度
* 84 柱状图最大矩形

---

# 4. Queue（队列）

使用 `collections.deque`

```python
from collections import deque

q = deque()
```

## 操作

```python
q.append(x)

q.popleft()

q[0]
```

## 复杂度

| 操作      | 时间   |
| ------- | ---- |
| append  | O(1) |
| popleft | O(1) |

---

# 5. Deque（双端队列）

```python
from collections import deque

dq = deque()
```

## 操作

```python
dq.append(x)

dq.appendleft(x)

dq.pop()

dq.popleft()
```

## 高频题

* 单调队列
* 滑动窗口最大值（239）

---

# 6. Set（哈希集合）

用于去重和快速查询。

```python
st = set()
st = set(nums)  # 从列表初始化哈希集合
st.clear()  # 清空哈希集合
```

## 操作

```python
st.add(x)

st.remove(x)

st.discard(x)   # 不存在时不报错

x in st
```

## 复杂度

| 操作     | 时间   |
| ------ | ---- |
| add    | O(1) |
| remove | O(1) |
| 查询     | O(1) |

---

# 7. Dict（哈希表）

最重要的数据结构之一。

```python
mp = {}
```

## 操作

```python
mp[key] = value

mp.get(key, 0)  # 获取 key 的值，key 不存在时返回默认值 0

for k in mp:
for k, v in mp.items():
for v in mp.values():
for i, v in enumerate(nums):
del mp[key]
```

遍历：

```python
for k, v in mp.items():
    print(k, v)
```

---

# 8. Counter（计数器）

```python
from collections import Counter

cnt = Counter(s)
```

## 示例

```python
cnt['a']
```

统计次数：

```python
Counter("aabbcc")
```

结果：

```python
{
'a':2,
'b':2,
'c':2
}
```

## 高频题

* 242 有效字母异位词
* 49 字母异位词分组

---

# 9. defaultdict

自动初始化。

```python
from collections import defaultdict

mp = defaultdict(int)
```

## 示例

```python
mp['a'] += 1
```

无需判断是否存在。

---

## 高频写法

### List

```python
mp = defaultdict(list)

mp[key].append(val)
```

### Set

```python
mp = defaultdict(set)

mp[key].add(val)
```

---

# 10. Heap（优先队列）

Python 使用 `heapq`

默认最小堆。

```python
import heapq

heap = []
```

## 操作

```python
heapq.heappush(heap, x)

heapq.heappop(heap)
heapq.heappop(heap)[1] # 如果堆中存储的是 (priority, value) 这样的元组，可以通过 [1] 获取 value
heap[0]
```

---

## 最大堆

```python
heapq.heappush(heap, -x)

x = -heapq.heappop(heap)
```

---

## 高频题

* 215 数组第K大元素
* 347 前K高频元素
* 295 数据流中位数

---

# 11. OrderedDict（很少）

```python
from collections import OrderedDict
```

LeetCode 高频：

* LRU Cache

不过 Python3.7+：

```python
dict
```

本身就保持插入顺序。

---

# 12. TreeNode（二叉树）

LeetCode 内置：

```python
class TreeNode:
    def __init__(self,val=0,left=None,right=None):
        self.val = val
        self.left = left
        self.right = right
```

访问：

```python
root.val

root.left

root.right
```

---

# DFS模板

```python
def dfs(root):
    res = [] # 存储结果的列表，可以根据需要修改
    if not root:    
        return
    # res.append(root.val) # 前序，用于复制树等
    dfs(root.left)
    # res.append(root.val) # 中序，用于二叉搜索树
    dfs(root.right)
    # res.append(root.val) # 后序，用于删除节点等
```

---

# BFS模板

```python
from collections import deque

q = deque([root])

while q:

    node = q.popleft()  # 访问节点，可以在这里处理 node.val

    if node.left:
        q.append(node.left)

    if node.right:
        q.append(node.right)
```

---

# 13. ListNode（链表）

LeetCode 内置：

```python
class ListNode:
    def __init__(self,val=0,next=None):
        self.val = val
        self.next = next
```

---

## 遍历

```python
cur = head

while cur:

    print(cur.val)

    cur = cur.next
```

---

## 虚拟头节点

```python
dummy = ListNode(0)

cur = dummy
```

高频必背。

---

# 14. Graph（图）

通常用邻接表。

```python
graph = defaultdict(list)
```

建图：

```python
for u,v in edges:
    graph[u].append(v)
```

---

## DFS

```python
def dfs(node):

    if node in visited:
        return

    visited.add(node)

    for nxt in graph[node]:
        dfs(nxt)
```

---

## BFS

```python
q = deque([start])

visited = {start}

while q:

    node = q.popleft()

    for nxt in graph[node]:

        if nxt not in visited:

            visited.add(nxt)

            q.append(nxt)
```

---

# 15. Trie（前缀树）

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.isEnd = False
```

---

## 插入

```python
node = root

for c in word:

    if c not in node.children:
        node.children[c] = TrieNode()

    node = node.children[c]

node.isEnd = True
```

---

## 高频题

* 208 Trie
* 211 Word Dictionary
* 212 Word Search II

---

# 16. Union Find（并查集）

```python
parent = list(range(n))
```

## find

```python
def find(x):

    if parent[x] != x:
        parent[x] = find(parent[x])

    return parent[x]
```

---

## union

```python
def union(x,y):

    px = find(x)
    py = find(y)

    if px != py:
        parent[px] = py
```

---

## 高频题

* 200 岛屿数量
* 684 冗余连接
* 547 省份数量

---

# 面试最重要的数据结构（出现率排序）

| 优先级   | 数据结构        | 必须掌握 |
| ----- | ----------- | ---- |
| ⭐⭐⭐⭐⭐ | List        | √    |
| ⭐⭐⭐⭐⭐ | Dict        | √    |
| ⭐⭐⭐⭐⭐ | Set         | √    |
| ⭐⭐⭐⭐⭐ | Stack       | √    |
| ⭐⭐⭐⭐⭐ | Queue/Deque | √    |
| ⭐⭐⭐⭐⭐ | TreeNode    | √    |
| ⭐⭐⭐⭐  | Heap        | √    |
| ⭐⭐⭐⭐  | Counter     | √    |
| ⭐⭐⭐⭐  | ListNode    | √    |
| ⭐⭐⭐⭐  | Graph       | √    |
| ⭐⭐⭐   | Trie        | √    |
| ⭐⭐⭐   | Union Find  | √    |

## 建议背熟的导入模板

```python
from collections import deque
from collections import Counter
from collections import defaultdict

import heapq
```

-----------
# python acm 模板
1. single int
``` python
n = int(input())
print(n)
```
2. tuple of int
``` python
n, m = map(int, input().split())
print(n, m)
```
3. list of int
``` python
n = int(input())
arr = list(map(int, input().split()))
arr.sort()
print(arr)
print(''.join(map(str, arr)))
```
4. matrix
``` python
n, m = map(int, input().split())
matrix = []
for i in range(n):
    row = list(map(int, input().split()))
    matrix.append(row)

for row in matrix:
    print(' '.join(map(str, row)))
```

5. multiple lines of input
``` python
T = int(input())
for _ in range(T):
    n = int(input())
    arr = list(map(int, input().split()))
    print(sum(arr))
```
6. string
``` python
s = input()
print(s)
print(s[::-1])  # reverse string
```
7. graph input
``` python
n, m = map(int, input().split()) # number of nodes and edges
graph = [[] for _ in range(n)]   # adjacency list
for _ in range(m): # read edges
    u, v = map(int, input().split()) # edge from u to v
    graph[u].append(v)
    graph[v].append(u)  # if undirected

for i in range(n):
    print(f"Node {i}: {graph[i]}")
```
---------------

# python 常用技巧
1. Indexing & Slicing
``` python
path = "usr/local/bin"
print(path[0])        # 'u'
print(path[4:9])      # 'local'
print(path[:])        # 'usr/local/bin'
print(path[::-1])     # 'nib/lacol/rsu'
print(path[:-1])      # 'usr/local/bi'
```
2. 数组初始化和遍历
``` python
n = 5
used = [False] * n  # 初始化长度为 n 的布尔数组
arr = [0] * n  # 初始化长度为 n 的数组
for i in range(n):#range(n) 生成 0 到 n-1 的整数序列
    arr[i] = i * i  # 填充数组
print(arr)  # 输出 [0, 1, 4, 9, 16]
for i in range(n,-1,-1): # 逆序遍历, -1 表示步长为 -1, -1 表示从 n-1 到 0
    print(i)  # 输出 5, 4, 3, 2, 1, 0
```

