---
title: 子序列问题
date: 2025-11-20 00:00:00 +0800
categories: [algorithm, dynamic-programming]
tags: [algorithm, dynamic programming]
author: ahern
math: true
---
## 子序列问题思路总结

### 思路 1：一维 dp 数组
```go
func dp(nums []int) int {
	// dp[i]表示以nums[i]结尾的最长递增子序列的长度， 包括nums[i]
	// note: 不是前i个元素的最长递增子序列的长度
	dp := make([]int, len(nums))	
	for i := 1; i <len(nums); i++ {
		for j := 0; j < i; j++ {
			if nums[i] > nums[j] {
				dp[i] = 最值(dp[i], dp[j]+1)
			}
		}
	}
}
```

- dp[i]表示以nums[i]结尾的最长递增子序列的长度，因为需要用到 nums[i] 做比较
- 如：最长递增子序列-[300](https://leetcode.cn/problems/longest-increasing-subsequence/)

### 思路 2：二维 dp 数组
```go
func dp(nums1 []int, nums2 []int) int {
	for i := 1; i <= len(nums1); i++ {
		for j := 1; j <=len(nums2); j++ {
			if nums1[i-1]==nums2[j-1]{
				dp[i][j]=dp[i-1][j-1]+1
			}else{
				dp[i][j]=最值(dp[i-1][j],dp[i][j-1])
			}
		}
	}
	return dp[len(nums1)][len(nums2)] 
}
```

- dp[i][j]表示nums1[0...i], nums2[0...j]最长公共子序列
- 如：最长公共子序列-[1143](https://leetcode.cn/problems/longest-common-subsequence/)

## 最长连续递增序列-[674](https://leetcode.cn/problems/longest-continuous-increasing-subsequence/)

设dp[i]表示以nums[i]结尾的最长递增子序列的长度（包括nums[i]），**且要求连续递增序列**
> Note: 最长连续递增子序列的长度不一定包含最后一个元素，所以需要一个变量记录最大值

```go
func findLengthOfLCIS(nums []int) int {
	if len(nums) <= 1 {
		return len(nums)
	}

	maxLength := 1
	dp := make([]int, len(nums)) // dp[i]表示以nums[i]结尾的最长连续递增子序列的长度
	dp[0] = 1 // 初始条件
	for i := 1; i < len(nums); i++ {
		if nums[i] > nums[i-1] {
			dp[i] = dp[i-1] + 1
		} else {
			// 初始条件
			dp[i] = 1
		}

		// 最长连续递增子序列的长度不一定包含最后一个元素，所以需要比较
		if dp[i] > maxLength {
			maxLength = dp[i]
		}
	}

	return maxLength
}
```

## 最长递增子序列-[300](https://leetcode.cn/problems/longest-increasing-subsequence/)
设dp[i]表示以nums[i]结尾的最长递增子序列的长度（包括nums[i]）
> dp[i]不是前i个元素的最长递增子序列的长度, 为什么？
>
> 答：因为需要比较递增，如果dp[i]表示前i个元素的最长递增子序列的长度，并不知道最后一个元素是哪一个

要求 dp[i], 也就是找到最大的 dp[j]（0 <= j < i）且 nums[i] > nums[j]。

需要注意的是：并不是 dp 数组最后一个元素为最终结果，应该最大子序列不一定包含最后一个元素。
```go
func lengthOfLIS(nums []int) int {
	// dp[i]表示以nums[i]结尾的最长递增子序列的长度， 包括nums[i]
	// note: 不是前i个元素的最长递增子序列的长度
	dp := make([]int, len(nums))
	for i := 0; i < len(nums); i++ {
		dp[i] = 1 // 初始化，每个元素本身就是一个递增子序列
	}
	
	maxLength := dp[0]
	for i := 1; i <len(nums); i++ {
		for j := 0; j < i; j++ {
			if nums[i] > nums[j] {
				dp[i] = max(dp[i], dp[j]+1)
			}
		}

		// 最长递增子序列的长度不一定包含最后一个元素，所以需要比较
		if dp[i] > maxLength {
			maxLength = dp[i]
		}
	}

	return maxLength
}
```

## 最长重复子数组-[718](https://leetcode.cn/problems/maximum-length-of-repeated-subarray/)
用指针 i 遍历数组 A，指针 j 遍历数组 B，当A[i] == B[j]时，说明存在公共子序列了，还需要看各自前一个元素的最长公共子序列（i-1、 j-1）。

设dp[i][j] 表示 A 数组以 i-1（不用纠结 i-1, 临界问题） 处结尾，B 数组以 j-1 处结尾的最长公共子序列长度。
当 A[i] == B[j] 时；此时出现公共子序列，到底有多少个呢？还得看看就要看各自前一个元素的最长公共子序列长度。
得出状态转移方程：
$$
dp[i][j] = dp[i-1][j-1] + 1
$$

```go
// dp[i][j] 表示 A 数组以 i-1 处结尾，B 数组以 j-1 处结尾的最长公共子序列长度
	dp := make([][]int, len(nums1)+1)
	for i := range dp {
		dp[i] = make([]int, len(nums2)+1)
	}

	result := 0
	// 遍历数组 A
	for i := 1; i < len(nums1)+1; i++ {
		// 遍历数组 B
		for j := 1; j < len(nums2)+1; j++ {
			// 如何果两个元素相等，就要看各自前一个元素的最长公共子序列长度，加 1 即可
			if nums1[i-1] == nums2[j-1] {
				dp[i][j] = dp[i-1][j-1] + 1
			}

			// 最长公共子序列不一定在 dp[len(nums1)][len(nums2)]，需要随时记录最大值
			if dp[i][j] > result {
				result = dp[i][j]
			}
		}
	}

	return result
```

## ？最长公共子序列-[1143](https://leetcode.cn/problems/longest-common-subsequence/)
设 dp[i][j] 表示 text1 在 0～i 范围，text2 在 0～j 范围的最长公共子序列。


有两种情况：
1、当text1[i] == text2[j]；表明该字符属于公共子序列，则 dp[i][j] = dp[i-1][j-1] + 1。（不理解？？）
<br>
2、当text1[i] != text2[j]；取 dp[i-1][j] 和 dp[i][j-1] 最大值


状态转移方程：

$$
dp[i][j] =
\begin{cases}
dp[i-1][j-1] + 1, & \text{if } text1[i] == text2[j] \\[6pt]
max(dp[i-1][j], dp[i][j-1]), & \text{if } text1[i] != text2[j] \\[6pt]
\end{cases}
$$

```go
func longestCommonSubsequence(text1 string, text2 string) int {
	t1 := len(text1)
	t2 := len(text2)
	dp:=make([][]int,t1+1)
	for i:=range dp{
		dp[i]=make([]int,t2+1)
	}

	for i := 1; i <= t1; i++ {
		for j := 1; j <=t2; j++ {
			if text1[i-1]==text2[j-1]{
				dp[i][j]=dp[i-1][j-1]+1
			}else{
				dp[i][j]=max(dp[i-1][j],dp[i][j-1])
			}
		}
	}
	return dp[t1][t2]
}
```

## 不相交的线-[1035](https://leetcode.cn/problems/uncrossed-lines/)
nums1[i] == nums2[j] 就可以连线；且要求连线不能相交，在 nums1、nums2 中寻找相同字符的相对顺序一致，这样就不会相交。

所以这也是一道求[最长公共子序列](https://leetcode.cn/problems/longest-common-subsequence/)

```go
func maxUncrossedLines(nums1 []int, nums2 []int) int {
    dp := make([][]int, len(nums1) + 1)
    for i := range dp {
        dp[i] = make([]int, len(nums2) + 1)
    }

	for i := 1; i <= len(nums1); i++ {
		for j := 1; j <= len(nums2); j++ {
			if (nums1[i - 1] == nums2[j - 1]) {
				dp[i][j] = dp[i - 1][j - 1] + 1
			} else {
				dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
			}
		}
	}
	
	return dp[len(nums1)][len(nums2)]
}
```

## 最大子序和-[53](https://leetcode.cn/problems/maximum-subarray/)

题目要点：求最大和的连续子数组。

设 dp[i] 表示以 nums[i] 结尾的最大连续子序列和。
在计算 dp[i] 时，就是需要考虑 nums[i] 是否需要与前面子序列拼接（dp[i-1]）,也就是：
```
if nums[i] + dp[i-1] > nums[i] {
	dp[i] = nums[i] + dp[i-1]
} 
```

最大和的连续子数组不一定包括最后一个元素，所以需要一个变量记录最大值

```go
func maxSubArray(nums []int) int {
	if len(nums) == 0 {
		return 0
	}

	// dp[i] 表示以 nums[i] 结尾的最大连续子序列和
	dp := make([]int, len(nums))
	for i := 0; i < len(nums); i++ {
		// 初始化条件；不拼接之前的子序列
		dp[i] = nums[i]
	}

	result := dp[0]
	for i := 1; i < len(nums); i++ {
		// 考虑 nums[i] 是否需要与前面子序列拼接（dp[i-1]）
        if nums[i] + dp[i-1] > nums[i] {
			dp[i] = nums[i] + dp[i-1]
		} 

		// 记录最大结果
		if dp[i] > result {
			result = dp[i]
		}
	}

	return result
}
```