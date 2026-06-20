## 回溯算法
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


## 动态规划

```
// 1. 定义状态：dp[i] = ...  // dp 数组的含义
// 2. 定义状态转移方程：dp[i] = ...  // 根据之前的状态转移到当前状态
// 3. 初始化：dp[0] = ...  // 根据状态定义初始化
// 4. 计算顺序：for (int i = 1; i < n; i++) { ... }  // 根据状态转移方程计算 dp 数组
```

## 滑动窗口
**极简模板：**
```python
l=0; 
for r in range(n): 
    更新窗口数据； 
    while 窗口不满足条件: 
        更新数据并 l++, 直到满足条件 
    更新答案
```