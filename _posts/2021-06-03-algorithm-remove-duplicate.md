---
layout: post
title: "初级算法: 删除排序数组中的重复项"
date:  2021-06-03 10:01:08
author: "Viletyy"
header-style: text
tags:
  - 算法
---
## 删除排序数组中的重复项

给你一个有序数组`nums`，请你原地删除重复出现的元素，使每个元素，只出现一次，返回删除后数组的新长度。

不要使用额外的数组空间，你必须在原地修改输入数组，并在使用O(1)额外空间的条件下完成。

### 说明
为什么返回数值是证书，但输出的答案是数组呢？

请注意，输入数组是以**引用**方式传递的，这意味着在函数里修改输入数组对于调用者是可见的。

你可以想象内部操作如下：
```go
// nums 是以引用方式传递的，也就是说，不对实参做任何拷贝。
len := removeDuplicates(nums);

// 在函数里修改输入数组对于调用者说可见的。
// 根据你的阐述返回的长度，它会打印出数组中该长度范围内的所有元素。
for i := 0; i < len; i++ {
    fmt.Println(nums[i])
}
```

### 示例1
输入：nums = [1, 1, 2]

输出：2, nums = [1, 2]

解释：函数应该返回新的长度2，并且原数组nums的前两个元素被修改为1，2，不需要考虑数组中超出新长度后面的元素

### 示例2
输入：nums = [0,0,1,1,1,2,2,3,3,4]

输出：5, nums = [0,1,2,3,4]

解释：函数应该返回新的长度 5 ， 并且原数组 nums 的前五个元素被修改为 0, 1, 2, 3, 4 。不需要考虑数组中超出新长度后面的元素。

### 题解
```go
func removeDuplicates(nums []int) int {
    if len(nums) == 0 {
        return 0
    }
    
    nui, i := 0, 1
    for ; i < len(nums); i++ {
        if nums[nui] == nums[i] {
           continue
        }
        nui++
        nums[nui] = nums[i]
    }
    return nui + 1
}
```