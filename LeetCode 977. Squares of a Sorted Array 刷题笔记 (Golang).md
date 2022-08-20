# 977. 有序数组的平方

*[LeetCode链接](https://leetcode.cn/problems/squares-of-a-sorted-array/)*

### 题目大意

给一个已经有序的数组，返回的数组也必须是有序的，且数组中的每个元素是由原数组中每个数字的平方得到的

进阶：设计时间复杂度为 O(n) 的算法解决本问题

### 解题思路

这一题由于原数组是**有序**的，所以要尽量利用这一特点来减少时间复杂度

定义一个新数组 target，和数组 nums 一样的大小，让 i 指向 target 数组终止位置。最终返回的数组 target，最后一位是最大值，这个值应该是由原数组最大值，或者最小值 (负数) 得来的，所以可以从数组的最后一位开始排列最终数组

考虑双指针法，用 2 个指针 left, right 分别指向原数组的首尾，分别计算平方值，然后比较两者大小，大的放在最终数组target的后面。然后大的那个指针移动，直至两个指针相撞 (相等)，最终数组就排列完成了

### 代码实现
```go
// 双指针法，时间复杂度O(n)
// 注意：
// 可以说使用额外数组target存储结果，空间复杂度O(n)
// 也可以说除了存储答案的数组以外，只需要维护常量空间，空间复杂度：O(1)
func sortedSquares(nums []int) []int {
	left, right, i := 0, len(nums)-1, len(nums)-1
	// var target [i+1]int   // 报错non-constant array bound i + 1
	target := make([]int, len(nums)) // 注意要用make声明，长度为i+1不是i
	for left <= right {
		if nums[left]*nums[left] < nums[right]*nums[right] {
			target[i] = nums[right] * nums[right]
			i--
			right--
		} else {
			target[i] = nums[left] * nums[left]
			i--
			left++
		}
	}
	return target
}
```

> 不能不使用target而只在原数组上操作的原因：
> 当第一次比较时，如果最左边的胜出，那么nums[n-1]就被nums[0]^2覆盖掉了
> 第二次比较时，应该是nums[1]和nums[n-1]比较，但此时的nums[n-1]就不是之前的那个数了，被覆盖掉了

常规解法：最简单的方法就是将数组 nums 中的数平方后直接排序 O(log n)

```go
// 解法2：暴力(不推荐)，平方后快排，时间复杂度O(n+log n)->O(log n)
func sortedSquares1(nums []int) []int {
	for i, value := range nums {
		nums[i] = value * value
	}
	sort.Ints(nums)
	return nums
}
/* 
对于[]int, []float, []string这种元素类型是基础类型的切片使用sort包提供的下面几个函数进行排序
    sort.Ints
    sort.Floats
    sort.Strings 
*/
```
最初想在一个数组上比较的错误记录：

```go
/* 思路完全错误，比较过程可能会进行多次平方运算，结果和预期平方排序不同
func sortedSquares(nums []int) []int {
    slow := 0
    for fast := 1; fast < len(nums); fast++ {
        if nums[fast]*nums[fast] < nums[slow]*nums[slow]{
            nums[slow], nums[fast] = nums[fast]*nums[fast], nums[slow]*nums[slow]
            // fast++
        } else {
            slow++
        }
    }
    return nums
} */
```

**注意1：**

* 不必太在意leetcode上执行用时，打败多少多少用户，这个就是一个玩具，非常不准确，一样的代码多提交几次可能就击败100%了

* 做题的时候自己能分析出来时间复杂度就可以了，至于 LeetCode 上执行用时，大概看一下就行，只要达到**最优的时间复杂度**就可以了


**注意2：**

* 可以说使用额外数组target存储结果，空间复杂度O(n)

* 也可以说除了存储答案的数组以外，只需要维护常量空间，空间复杂度：O(1)

**注意3：**

* 要用 **make** 声明 target：`target := make([]int, len(nums))`
* `var target [len(nums)]int`或 `var target [i+1]int` 会报错 `non-constant array bound len(nums)或i + 1`