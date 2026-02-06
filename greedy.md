# 贪心算法
## 买卖股票的最佳时机 II: 多次买卖
输入一个数组，它的第 i 个元素是一支给定股票第 i 天的价格：`prices = [7,1,5,3,6,4]` 。

### 关键

`profit = price[j] - price[i]`，其中 `j > i`。

又， `profit = price[j] - price[i] = price[j] - price[j-1] + price[j-1] - price[j-2] + ... + price[i+1] - price[i]`。

最大利润等于所有正的 `price[j] - price[i]` 的和。

### 代码

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int max_profit = 0;
        for (int i = 1; i < prices.size(); i++) {
            if (prices[i] > prices[i - 1]) {
                max_profit += prices[i] - prices[i - 1];
            }
        }
        return max_profit;
    }
};
```
## 买卖股票的最佳时机 I：一次买卖
输入一个数组，它的第 i 个元素是一支给定股票第 i 天的价格：`prices = [7,1,5,3,6,4]` 。

### 关键
局部最优：过去天数中的最低价格已确定，每到新的一天，比较当前价格与最低价格的差值，更新最大利润。

### 代码

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int min_price = 0;
        int result = 0;
        for (int i = 1; i < prices.size(); i++) {
            if (prices[i-1] < prices[min_price]) {
                min_price = i - 1;
            }
            result = max(result, prices[i] - prices[min_price]);
        }
        return result;
    }
};
```

