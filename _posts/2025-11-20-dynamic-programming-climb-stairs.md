---
title: 动态规划-爬楼梯问题
date: 2025-11-20 00:00:00 +0800
categories: [algorithm, dynamic-programming]
tags: [algorithm, dynamic programming]
author: ahern
math: true

---

## 爬楼梯-70

### 题目描述

假设你正在爬楼梯。需要 n 阶你才能到达楼顶。

每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？

```
示例 1：

输入：n = 2
输出：2
解释：有两种方法可以爬到楼顶。
1. 1 阶 + 1 阶
2. 2 阶
   示例 2：

输入：n = 3
输出：3
解释：有三种方法可以爬到楼顶。
1. 1 阶 + 1 阶 + 1 阶
2. 1 阶 + 2 阶
3. 2 阶 + 1 阶


提示：

1 <= n <= 45
```

### 解题思路

设$S(n)$表示爬到第n阶楼梯的方法数，n-1处，可以再爬1阶可以登顶, 有$S(n-1)$种方方式；
n-2处，可以再爬2阶也可以登顶，有$S(n-2)$种方式。因此也是斐波那契问题，状态转移方程为：

$$
S(n) =
\begin{cases}
1, & \text{if } n = 1 \\[6pt]
2, & \text{if } n = 2 \\[6pt]
S(n-1) + S(n-2), & \text{if } n > 2
\end{cases}
$$

```go
func climbStairs(n int) int {
    if n == 1 {
    return 1
    }

    dp := make([]int, n+1)
    dp[1] = 1
    dp[2] = 2

    for i := 3; i < n+1; i++ {
    dp[i] = dp[i-1] + dp[i-2]
    }

    return dp[n]
}
```

## 组合总和IV-377

### 题目描述
给你一个由 不同 整数组成的数组 nums ，和一个目标整数 target 。请你从 nums 中找出并返回总和为 target 的元素组合的个数。

题目数据保证答案符合 32 位整数范围。


```
示例 1：

输入：nums = [1,2,3], target = 4
输出：7
解释：
所有可能的组合为：
(1, 1, 1, 1)
(1, 1, 2)
(1, 2, 1)
(1, 3)
(2, 1, 1)
(2, 2)
(3, 1)
请注意，顺序不同的序列被视作不同的组合。
示例 2：

输入：nums = [9], target = 3
输出：0


提示：

1 <= nums.length <= 200
1 <= nums[i] <= 1000
nums 中的所有元素 互不相同
1 <= target <= 1000
```

进阶：如果给定的数组中含有负数会发生什么？问题会产生何种变化？如果允许负数出现，需要向题目中添加哪些限制条件？

### 解题思路
根据题意，顺序不同视为不同组合，其实是排列问题。

也可以理解成爬楼梯问题，假设楼梯有 target 阶，每次可以爬 nums 中的任意阶数，问有多少种不同方式爬到顶？

类比到【爬楼梯-70】，nums为[1,2]。

dp[i]表示凑成 i 的排列数。临界条件dp[0] = 1, 当 i==nums[j]时，i-nums[j]=0。表示仅nums[j]即可凑成 i，故 dp[0]=1

```go
func combination(nums []int, target int) int {
	// dp[i]表示凑成 i 的排列数
	dp := make([]int, target+1)

	// 临界条件，当 i==nums[j]时，i-nums[j]=0。表示仅nums[j]即可凑成 i，故 dp[0]=1
	dp[0] = 1
	for i := 1; i < target+1; i++ {
		// 依次尝试每个数
		for j := 0; j < len(nums); j++ {
			if i >= nums[j] {
				// 右dp[i]表示前 j-1 个数凑成 i 的排列数。
				// dp[i-nums[j]]表示在i-nums[j]处，加上nums[j]刚好凑成 i
				dp[i] = dp[i] + dp[i-nums[j]]
			}
		}
	}

	return dp[target]
}
```

## 零钱兑换-322
### 题目描述
给你一个整数数组 coins ，表示不同面额的硬币；以及一个整数 amount ，表示总金额。

计算并返回可以凑成总金额所需的 最少的硬币个数 。如果没有任何一种硬币组合能组成总金额，返回 -1 。

你可以认为每种硬币的数量是无限的。


```
示例 1：

输入：coins = [1, 2, 5], amount = 11
输出：3
解释：11 = 5 + 5 + 1
示例 2：

输入：coins = [2], amount = 3
输出：-1
示例 3：

输入：coins = [1], amount = 0
输出：0


提示：

1 <= coins.length <= 12
1 <= coins[i] <= 231 - 1
0 <= amount <= 104
```

### 解题思路
也是一个爬楼梯问题，爬到 i 阶楼梯，每次可以爬 coins[j] 阶数，求最少跳数？
> 如何体现出重复爬coins[j]阶？
> <br>
> 因为每爬到第 i 阶梯，都会从 coins 中取出 coins[j] 阶数，提现了coins[j]重复使用。

设 dp[i]表示爬到 i 阶楼梯的最少跳数，则可以从可以从dp[i-coins[0]]、dp[i-coins[1]]、dp[i-coins[2]] ...位置跳上来，跳数 + 1。取最小值。

转移方程：
$$
dp[i] = 
\begin{cases}
0, & \text{if } i = 1 \\[6pt]
min(dp[i], dp[i-coins[i]] + 1) & \text{if } i >= coins[i]
\end{cases}
$$

临界条件 dp[0]: i == coins[j]；在 0 阶梯处，跳coins[j]即可登顶，则dp[0] + 1 = 1

```go
func coinChange(coins []int, amount int) int {
	dp := make([]int, amount+1)
	dp[0] = 0 // i == coins[j]；dp[0] + 1 = 1次就可以登顶

	for i := 1; i < amount+1; i++ {
		dp[i] = math.MaxInt32
		for j := 0; j < len(coins); j++ {
			if i >= coins[j] {
				dp[i] = min(dp[i], dp[i-coins[j]]+1)
			}
		}

		fmt.Println(dp)
	}

	if dp[amount] == math.MaxInt32 {
		return -1
	}

	return dp[amount]
}
```

## 使用最小花费爬楼梯-746

### 题目描述

给你一个整数数组 cost ，其中 cost[i] 是从楼梯第 i 个台阶向上爬需要支付的费用。一旦你支付此费用，即可选择向上爬一个或者两个台阶。

你可以选择从下标为 0 或下标为 1 的台阶开始爬楼梯。

请你计算并返回达到楼梯顶部的最低花费。

```
示例 1：

输入：cost = [10,15,20]
输出：15
解释：你将从下标为 1 的台阶开始。
- 支付 15 ，向上爬两个台阶，到达楼梯顶部。
  总花费为 15 。

示例 2：

输入：cost = [1,100,1,1,1,100,1,1,100,1]
输出：6
解释：你将从下标为 0 的台阶开始。
- 支付 1 ，向上爬两个台阶，到达下标为 2 的台阶。
- 支付 1 ，向上爬两个台阶，到达下标为 4 的台阶。
- 支付 1 ，向上爬两个台阶，到达下标为 6 的台阶。
- 支付 1 ，向上爬一个台阶，到达下标为 7 的台阶。
- 支付 1 ，向上爬两个台阶，到达下标为 9 的台阶。
- 支付 1 ，向上爬一个台阶，到达楼梯顶部。
  总花费为 6 。


提示：

2 <= cost.length <= 1000
0 <= cost[i] <= 999
```

### 解题思路

设$S(n)$表示到达第n阶楼梯的最小花费，则可以在n-1阶楼梯处，支付`cost[n-1]`，再爬1阶到达n阶；
也可以在n-2阶楼梯处，支付`cost[n-2]`，再爬2阶到达n阶。取两者的最小值。
因此状态转移方程为：

$$
S(n) =
\begin{cases}
0, & \text{if } n = 0 \\[6pt]
0, & \text{if } n = 1 \\[6pt]
min(S(n-1) + cost[n-1], S(n-2) + cost[n-2]), & \text{if } n > 2
\end{cases}
$$

关键点：

- 可以选择从下标为 0 或下标为 1 的台阶开始爬楼梯；意味着下标为 0 或下标为 1 的台阶花费为 0。

- 下标为 len(cost) 是楼顶，而不是 len(cost)-1。所以登顶最低费用为 dp[len(cost)]。
  
  | cost | 1   | 100 | 1   | 1   | 1   | 100 | 1   | 1   | 100 | 1   | <mark>楼顶</mark> |
  | ---- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --------------- |
  | 下标   | 0   | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9   | 10              |
  | dp数组 | 0   | 0   | 1   | 2   | 2   | 3   | 3   | 4   | 4   | 5   | 6               |

```go
func minCostClimbingStairs(cost []int) int {
    dp := make([]int, len(cost)+1) // 第i个台阶的最小花费，i从0开始；下标 len(cost) 代表楼顶

    // 可以选择从下标为 0 或下标为 1 的台阶开始爬楼梯，意味着初始花费为 0
    dp[0] = 0
    dp[1] = 0

    for i := 2; i < len(cost)+1; i++ {
        dp[i] = min(dp[i-1]+cost[i-1], dp[i-2]+cost[i-2])
    }

    // 下标 len(cost) 代表到达楼顶
    return dp[len(cost)]
}
```

## 完全平方数-279
### 题目描述
给你一个整数 n ，返回 和为 n 的完全平方数的最少数量 。

完全平方数 是一个整数，其值等于另一个整数的平方；换句话说，其值等于一个整数自乘的积。例如，1、4、9 和 16 都是完全平方数，而 3 和 11 不是。


```
示例 1：

输入：n = 12
输出：3
解释：12 = 4 + 4 + 4
示例 2：

输入：n = 13
输出：2
解释：13 = 4 + 9

提示：

1 <= n <= 104
```

### 解题思路
可以转化成爬楼梯问题，爬到n层， 每次可以从$[1*1, 2*2, 3*3,...j*j]$层跳上来，求最少跳跃次数。

设 $dp[i]$ 表示爬到第 i 层最少跳跃次数，可以从 $dp[i - j*j]$ 层跳上来，跳跃数 + 1，则有状态转移方程：

$$
dp[i] =
\begin{cases}
0, & \text{if } i = 0 \\[6pt]
min(dp[i], dp[i-j*j] + 1) & \text{if } i > 0
\end{cases}
$$

临界条件 $dp[0]$: `i == j*j`；在 0 阶梯处，跳 `j*j` 即可登顶，则$dp[0] + 1 = 1$

```go
func numSquares(n int) int {
	dp := make([]int, n+1)
	dp[0] = 0
	for i := 1; i < n+1; i++ {
		dp[i] = math.MaxInt32
	}

	for i := 1; i < n+1; i++ {
		// 相当与从[1*1, 2*2, 3*3,..., n*n]中取出阶数
		for j := 1; j <= n; j++ {
			nn := j * j

			if i >= nn {
				dp[i] = min(dp[i], dp[i-nn]+1)
			}
		}
	}

	return dp[n]
}
```
