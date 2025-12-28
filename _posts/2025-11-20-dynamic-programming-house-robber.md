---
title: 打家劫舍
date: 2025-11-20 00:00:00 +0800
categories: [algorithm, dynamic-programming]
tags: [algorithm, dynamic programming]
author: ahern
math: true
---

## 打家劫舍-[198](https://leetcode.cn/problems/house-robber/description/)

dp[i] 表示第 i 间房间的最大偷窃金额

对于一个房间 i，有偷窃和不偷窃两种方式：
<br>
1、偷窃：那么可以偷窃第 i-2 间，不可以偷窃第 i-1间，偷窃总金额为 $dp[i-2] + nums[i]$

2、不偷窃：那么不可以偷窃第 i-2 间，可以偷窃第 i-1间，偷窃总金额为 $dp[i-1]$

<br>
临界条件：

1、第1间房间; 最大偷窃金额为nums[0]

2、第2间房间; 最大偷窃金额为max(nums[0], nums[1])

<br>
状态转移方程：

$$
dp[i] =
\begin{cases}
nums[0], & \text{if } i == 0 \\[6pt]
max(nums[0], nums[1]), & \text{if } i == 1 \\[6pt]
max(dp[i-1], dp[i-1] + nums[i]), & \text{if } i >= 2 
\end{cases}
$$


```go
func rob(nums []int) int {
    if len(nums) == 0 {
        return 0
    }
    if len(nums) == 1 {
        return nums[0]
    }
    dp := make([]int, len(nums))
    dp[0] = nums[0]
    dp[1] = max(nums[0], nums[1])
    for i := 2; i < len(nums); i++ {
        dp[i] = max(dp[i-2] + nums[i], dp[i-1])
    }
    return dp[len(nums)-1]
}
```

## 打家劫舍II-[213](https://leetcode.cn/problems/house-robber-ii/description/)

与【打家劫舍】不同之处在于：第一个房屋和最后一个房屋是紧挨着。所有有两种打劫方式：
<br>
1、打劫第一个间到倒数第二间：nums[:len(nums)-1]

2、打劫第二个间到最后间：nums[1:]

然后取最大值。

```go
func rob(nums []int) int {
	if len(nums) == 1 {
		return nums[0]
	}
	if len(nums) == 2 {
		return max(nums[0], nums[1])
	}

	return max(
		// 打劫第1间 ～ 倒数第二间
		_rob(nums[:len(nums)-1]),
		// 打劫第2间 ～ 最后一间
		_rob(nums[1:]),
	)
}

func _rob(nums []int) int {
	if len(nums) == 1 {
		return nums[0]
	}
	if len(nums) == 2 {
		return max(nums[0], nums[1])
	}

	dp := make([]int, len(nums))
	dp[0] = nums[0]
	dp[1] = max(nums[0], nums[1])
	for i := 2; i < len(nums); i++ {
		dp[i] = max(dp[i-1], dp[i-2]+nums[i])
	}

	return dp[len(nums)-1]
}
```

## 打家劫舍II-[213](https://leetcode.cn/problems/house-robber-iii/description/)
todo

