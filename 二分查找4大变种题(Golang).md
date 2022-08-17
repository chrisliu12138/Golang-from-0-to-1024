## 通过LeetCode 34 总结二分查找4大变种题
### 34. Find First and Last Position of Element in Sorted Array

*LeetCode链接：[34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/ "leetcode.cn")*
### 题目大意
给定一个按照**非递减 (可以有多个相同元素)** 排列的整数数组 nums，和一个目标值 target，找出给定目标值在数组中的开始位置和结束位置，如果数组中不存在目标值，返回 [-1, -1]

算法时间复杂度必须是 O(log n) 级别 (不能暴力循环)


### 解题思路
给出一个有序数组 nums 和一个数 target，要求在数组中找到第一个和这个元素相等的元素下标，最后一个和这个元素相等的元素下标

这一题是**经典的二分搜索变种题**

二分搜索有 **4 大基础变种题：**

1. 查找第一个值等于给定值的元素
2. 查找最后一个值等于给定值的元素
3. 查找第一个大于等于给定值的元素
4. 查找最后一个小于等于给定值的元素

这一题的解题思路可以分别利用变种 1 和变种 2 的解法就可以做出此题

### 代码实现

```go
// 不要想着直接return左右两个边界 要用主函数调用两个函数分别return
func searchRange(nums []int, target int) []int {
	if len(nums) == 0 {
	    return []int{-1, -1}
	}   // 不确定该判断能不能提升性能
	return []int{searchLeftRange(nums, target), searchRightRange(nums, target)}
}

// 变种1：二分查找第一个与 target 相等的元素，时间复杂度 O(logn)
func searchLeftRange(nums []int, target int) int {
	left, right := 0, len(nums)-1
	for left <= right {
		mid := left + (right-left)/2
		if nums[mid] > target {
			// mid = right - 1 开始写错成修改mid
			right = mid - 1
		} else if nums[mid] < target {
			// mid = left + 1
			left = mid + 1
		} else {
			if mid == 0 || nums[mid-1] != target { // 找到第一个与target相等的元素直接return
				return mid
			} else {            // 不用else{}也可以
				right = mid - 1 // 这行是核心 一开始我写成mid--
			} 
		}
	}
	return -1
}

// 变种2：二分查找最后一个与 target 相等的元素，时间复杂度 O(logn)
func searchRightRange(nums []int, target int) int {
	left, right := 0, len(nums)-1
	for left <= right {
		mid := left + (right-left)/2
		if nums[mid] > target {
			right = mid - 1
		} else if nums[mid] < target {
			left = mid + 1
		} else {
			//if (mid == len(nums)-1) || (nums[mid+1] != target) {  // 优先级：+、- 大于 ==、!= 大于||，可以不加括号
			if mid == len(nums)-1 || nums[mid+1] != target { // 找到最后一个与target相等的元素直接return
				return mid
			} else {
				left = mid + 1 // 这行是与二分搜索裸题不同的核心！！
			}
		}
	}
	return -1
}
```

或者用一次变种 1 的方法，然后循环往后找到最后一个与给定值相等的元素。不过后者这种方法可能会使时间复杂度下降到 O(n)，因为有可能数组中 n 个元素都和给定元素相同 (**4 大基础变种的实现见代码**)

**变种题3，4代码：**

```go
// 变种3：二分查找第一个大于等于 target 的元素，时间复杂度 O(logn)
func searchFirstGreaterElement(nums []int, target int) int {
	left, right := 0, len(nums)-1
	for left <= right {
		mid := left + (right - left)/2
		if nums[mid] >= target {
			if (mid == 0) || (nums[mid-1] < target) { // 找到第一个大于等于 target 的元素
				return mid
			}
			right = mid - 1
		} else {
			left = mid + 1
		}
	}
	return -1
}

// 变种4：二分查找最后一个小于等于 target 的元素，时间复杂度 O(logn)
func searchLastLessElement(nums []int, target int) int {
	left, right := 0, len(nums)-1
	for left <= right {
		mid := left + (right - left)/2
		if nums[mid] <= target {
			if (mid == len(nums)-1) || (nums[mid+1] > target) { // 找到最后一个小于等于 target 的元素
				return mid
			}
			left = mid + 1
		} else {
			right = mid - 1
		}
	}
	return -1
}
```

**注意以下常见问题：**

```go
//注意同时return两个值的写法
return []int{-1, -1}
return []int{ans1, ans2}

//不确定该判断能不能提升性能
if len(nums) == 0 {
    return []int{-1, -1}
}  

//优先级：+、- 大于 ==、!= 大于||，可以不加括号
if (mid == len(nums)-1) || (nums[mid+1] != target) {  
if mid == len(nums)-1 || nums[mid+1] != target {
```
