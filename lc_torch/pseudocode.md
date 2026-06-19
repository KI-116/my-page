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

## 哈希表

需要快速查找或记录元素，或判断出现频次

> std::unordered_set / std::unordered_map 是 C++ 中最常用的哈希结构，底层实现是哈希表，查询 O(1)，增删O(1);遍历 O(n)。
1. unordered_set
```cpp
unordered_set<int> s; // 存储整数的哈希集合
unordered_set<int> nums(nums.begin(), nums.end()); // 从 vector 初始化哈希集合
unordered_set<int> s2(s); // 从另一个哈希集合初始化
unordered_set<int> s3 = {1, 2, 3}; // 列表初始化哈希集合
s.insert(5); // 插入元素
if (s.count(5))  // 判断元素是否存在
if (s.find(5) != s.end()) // 判断元素是否存在
s.erase(5); // 删除元素
```

``` python
s = set() # 创建一个空集合
s = set(nums) # 从列表初始化集合
s.add(5) # 插入元素
if 5 in s: # 判断元素是否存在
for x in s:
    print(x)
s.remove(5) # 删除元素
```

2. unordered_map
```cpp
std::unordered_map<std::string, int> m; // 存储字符串到整数映射的哈希映射
m["apple"] = 3; // 插入键值对
std::unordered_map<std::string, int> m2(m); // 从另一个哈希映射初始化
std::unordered_map<int, int> nums;
nums.insert({1, 100}); // 插入键值对
if (m.count("apple")) // 判断键是否存在
if (m.find("apple") != m.end()) // 判断键是否存在
m.erase("apple"); // 删除键值对
```

``` python
m = {} # 创建一个空字典
m = dict(zip(keys, values)) # 从两个列表初始化字典
m["apple"] = 3 # 插入键值对
if "apple" in m: # 判断键是否存在
    print(m["apple"]) # 访问键对应的值
for key, value in m.items():
    print(key, value)
del m["apple"] # 删除键值对
# unimportant
m.pop("apple", None) # 删除键值对，若键不存在则返回 None
m.clear() # 清空字典
m.get("apple", 0) # 获取键对应的值，若键不存在则返回默认值 0
m.keys() # 获取所有键
m.values() # 获取所有值
m.items() # 获取所有键值对
```

## 

``` python
result = []
