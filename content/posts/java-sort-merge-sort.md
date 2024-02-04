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

## 排序

leetcode: [912. 排序数组](https://leetcode.cn/problems/sort-an-array/)

### 归并排序

自顶向下：先分组，合并时比较大小

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

自底向下：默认是大小为一的数组，两两归并，四四归并，...，直到最后一次归并完成

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



参考题目：

[88. 合并两个有序数组](https://leetcode.cn/problems/merge-sorted-array/)

[LCR 170. 交易逆序对的总数](https://leetcode.cn/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)

