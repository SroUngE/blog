---

title: "排序：归并排序"

description: "归并排序的算法实现，基于《算法》的java版本，包括自顶向下和自底向上的两种。"

date: 2024-01-12

tags: ["Java", "Algorithm"]

featuredImage: "https://s2.loli.net/2024/01/23/3qPWMLs9UBogi7A.png"

featuredImagePreview: "https://s2.loli.net/2024/01/23/3qPWMLs9UBogi7A.png"

draft: false

---

<!--more-->

## 思路

归并排序的思路是，要将一个数组排序，可以先（递归地）将它分成两半分别排序，然后将结果归并起来。

时间复杂度：O(nlogn)

空间复杂度：O(n)

### 自顶向下的归并排序

自顶向下的归并排序是 **分治思想** 的典型应用，将一个大问题分割成小问题分别解决。

要对子数组 `nums[low ... high]` 进行排序，先将它分为 `nums[low .. mid]` 和 `nums[mid + 1, high]`两部分，分别通过递归调用将它们单独排序，最后将有序的子数组归并为最终的排序结果。

```java
private int[] temp;

public int[] sortArray(int[] nums) {
    temp = new int[nums.length];
    mergeSort(nums, 0, nums.length - 1);
    return nums;
}

/**
* 1. 先把数组分为nums[low, mid]和nums[mid + 1, high]；
* 2. 分别对子数组排序；
* 3. 合并排序后的子数组；
*/
public void mergeSort(int[] nums, int low, int high) {
    if (low >= high) returnl;
    int mid = low + (high - low) / 2;
    mergeSort(nums, low, mid);
    mergeSort(nums, mid + 1, high);
    merge(nums, low, mid, high);
}

/**
* 1. 左半边用尽，则取右半边的元素；
* 2. 右半边用尽，则取左半边的元素；
* 3. 右半边的元素 < 左半边的元素，则取右半边的元素；
* 4. 右半边的元素 >= 左半边的元素，则取左半边的元素；
*/
public void merge(int[] nums, int low, int mid, int high) {
    for (int i = low ; i <= high ; i++) {
        temp[i] = nums[i];
    }
    int left = low, right = mid + 1;
    for (int i = low ; i <= high ; i++) {
        if (left > mid) nums[i] = temp[right++];
        else if (right > high) nums[i] = temp[left++];
        else if (temp[right] < temp[left]) nums[i] = temp[right++];
        else nums[i] = temp[left++];
    }
}
```

### 自底向上的归并排序

自底向上的归并排序会多次遍历整个数组，根据子数组大小进行两两归并。

子数组的大小 `sz` 的初始值为1，每次加倍。两两归并，四四归并，...，直到最后一次归并完成。

```java
private int[] temp;

public int[] sortArray(int[] nums) {
    temp = new int[nums.length];
    mergeSort(nums, 0, nums.length - 1);
    return nums;
}

/**
* 1. sz 是子数组的大小，初始值为 1，成倍增加
* 2. 将当前数组分为若干个大小为sz的子数组，并合并
*/
public void mergeSort(int[] nums, int low, int high) {
    int n = nums.length;
    for (int sz = 1 ; sz < n ; sz = sz + sz) {
        for (int low = 0 ; low < n - sz ; low += sz + sz) {
            int mid = low + sz - 1;
            int high = Math.min(low + sz + sz -1, N-1);
            merge(nums, low, mid, high);
        }
    }
}
```

自底向上的归并排序比较适合用 **链表** 组织的数据。这种方法只需要重新组织链表链接就能将链表原地排序。

### 参考题目

[912. 排序数组](https://leetcode.cn/problems/sort-an-array/)

[88. 合并两个有序数组](https://leetcode.cn/problems/merge-sorted-array/)

[LCR 170. 交易逆序对的总数](https://leetcode.cn/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)
