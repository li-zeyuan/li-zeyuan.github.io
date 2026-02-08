---
title: 股票问题
date: 2021-11-23 00:00:00 +0800
categories: [algorithm, dynamic-programming]
tags: [algorithm, dynamic programming]
author: ahern
math: true
---

方法论：https://labuladong.online/algo/dynamic-programming/stock-problem-summary/

## 买卖股票的最佳时机-[121](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)
### 暴力方式

假设第 i(i>1) 天就卖出股票，则可以遍历 i 天之前的价格，找到最大的收益。

```go
func maxProfit(prices []int) int {
	if len(prices) <= 1 {
		return 0
	}

	result := 0
	for i := 1; i < len(prices); i++ { // 卖出
		for j := 0; j < i; j++ { // 买入
			if prices[i]-prices[j] > result {
				result = prices[i] - prices[j]
			}
		}
	}
	return result
}
```

### 贪心算法
低买高卖，就可以获得最大收益。依次遍历，找到左边最小值，计算最大收益

```go
func maxProfit(prices []int) int {
	if len(prices) <= 1 {
		return 0
	}

	low := prices[0] // 左边最低价格
	result := 0      // 最大收益
	for i := 1; i < len(prices); i++ {
		if prices[i] < low {
			low = prices[i]
		}

		if prices[i]-low > result {
			result = prices[i] - low
		}
	}

	return result
}
```

### 动态规划
设 dp[i][0]表示第 i 天手上没有股票最大收益，dp[i][1]表示第 i 天手上有股票最大收益。

dp[i][0]存在两种情况，取最大值：

$$
dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])
$$

1 、前一天手上没有股票，今天不操作
<br>
2 、前一天手上有股票，今天卖出

<br>
dp[i][1]存在两种情况，取最大值：

$$
dp[i][1] = max(dp[i-1][1], -prices[i])
$$

1 、前一天手上有股票，今天不操作
<br>
2 、今天买入

<br>
最结果，手没有股票收益最大

```go
func maxProfit(prices []int) int {
	dp := make([][2]int, len(prices))
	dp[0][0] = 0          // 第一天不持有股票的最大收益
	dp[0][1] = -prices[0] // 第一天持有股票的最大收益

	for i := 1; i < len(prices); i++ {
		// 第i天不持有股票的最大收益
		dp[i][0] = max(
			dp[i-1][0],           // 前一天不持有股票
			dp[i-1][1]+prices[i], // 前一天持有股票，今天卖出
		)

		// 第i天持有股票的最大收益
		dp[i][1] = max(
			dp[i-1][1], // 前一天持有股票
			-prices[i], // 今天买入
		)
	}

	return dp[len(prices)-1][0]
}
```

## 买卖股票的最佳时机II-[122](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)

与【买卖股票的最佳时机】不同的是，这道题可以多次买卖股票。第i天持有股票的最大收益为：

$$
dp[i][1] = max(dp[i-1][1], dp[i-1][0]-prices[i])
$$

 > Note: dp[i-1][0] i-1天不持有股票，可能已经进行了一轮交易

```go
func maxProfit(prices []int) int {
	dp := make([][2]int, len(prices))
	dp[0][0] = 0          // 第一天不持有股票的最大收益
	dp[0][1] = -prices[0] // 第一天持有股票的最大收益

	for i := 1; i < len(prices); i++ {
		// 第i天不持有股票的最大收益
		dp[i][0] = max(
			dp[i-1][0],           // 前一天不持有股票
			dp[i-1][1]+prices[i], // 前一天持有股票，今天卖出
		)

		// 第i天持有股票的最大收益
		dp[i][1] = max(
			dp[i-1][1],           // 前一天持有股票
			dp[i-1][0]-prices[i], // 前一天不持有股票，今天买入
		)
	}

	return dp[len(prices)-1][0]
}
```