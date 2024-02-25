---
title: "排序：快速排序"
date: 2024-02-24
draft: false
tags: ["Java", "Algorithm"]
featuredImage: "https://s2.loli.net/2024/01/23/3qPWMLs9UBogi7A.png"
featuredImagePreview: "https://s2.loli.net/2024/01/23/3qPWMLs9UBogi7A.png"
---

### 思路

快速排序是一种 分治 的排序算法。它将一个数组分成两个子数组，将两部分独立地排序。

> 快速排序与归并排序是互补的：归并排序将数组分成两个子数组分别排序，并将有序的子数组归并以整个数组排序；而快速排序则是当两个子数组都有序时整个数组也就自然有序了。在第一种情况中，递归调用发生在处理整个数组之前；在第二种情况中，递归调用发生在处理整个数组之后。在归并排序中，一个数组被等分为两半；在快速排序中，切分（partition）的位置取决于数组的内容。

时间复杂度：O(nlogn)

空间复杂度：辅助栈

### 代码

```java
public int[] sortArray(int[] nums) {
    quickSort(nums, 0, nums.length - 1);
    return nums;
}

public void quickSort(int[] nums, int low, int high) {
    if (high <= low) return;
    int j = partition(nums, low, high);
    quickSort(nums, low, j - 1);
    quickSort(nums, j + 1, high);
}

public int partition(int[] nums, int low, int high) {
    int i = low, j = high + 1;
    int pivot = nums[low];				// 选取 nums[low] 为基准元素
    while (i < j) {
        while (nums[++i] < pivot) { 	// 寻找比 pivot 小的元素
            if (i == high) break;
        }
        while (nums[--j] > pivot) { 	// 寻找比 pivot 大的元素
            if (j == low) break;
        }
        if (i < j) swap(nums, i, j);	// 交换元素
    }
    swap(nums, low, j);					// 将 pivot 放到中间
    return j;
}

private void swap(int[] nums, int i, int j) {
    int temp = nums[i];
    nums[i] = nums[j];
    nums[j] = temp;
}
```

### 随机数版

当前的基准总是取子数组的第一个元素，这样对于逆序的数组，时间复杂会是平方级别的，因此我们可以随机地取基准元素。

```java
Random random = new Random;

public int[] sortArray(int[] nums) {
    quickSort(nums, 0, nums.length - 1);
    return nums;
}

public void quickSort(int[] nums, int low, int high) {
    if (high <= low) return;
    int j = partition(nums, low, high);
    quickSort(nums, low, j - 1);
    quickSort(nums, j + 1, high);
}

public int partition(int[] nums, int low, int high) {
    // 从 [high, low] 里取一个随机数
    int r = random.nextInt(high - low + 1) + low;
    swap(nums, low, r);
    
    int pivot = nums[low];
    int i = low, j = high + 1;				
    while (i < j) {
        while (nums[++i] < pivot) { 	
            if (i == high) break;
        }
        while (nums[--j] > pivot) { 	
            if (j == low) break;
        }
        if (i < j) swap(nums, i, j);	
    }
    swap(nums, low, j);					
    return j;
}

private void swap(int[] nums, int i, int j) {
    int temp = nums[i];
    nums[i] = nums[j];
    nums[j] = temp;
}
```

### 参考题目

[912. 排序数组](https://leetcode.cn/problems/sort-an-array/)

[215. 数组中的第K个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array/)

[347. 前 K 个高频元素](https://leetcode.cn/problems/top-k-frequent-elements/)
