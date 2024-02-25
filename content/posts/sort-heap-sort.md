---
title: "排序：堆排序"
date: 2024-02-25
draft: false
tags: ["Java", "Algorithm"]
featuredImage: "https://s2.loli.net/2024/01/23/3qPWMLs9UBogi7A.png"
featuredImagePreview: "https://s2.loli.net/2024/01/23/3qPWMLs9UBogi7A.png"
---





### 代码

```java
public int[] sortArray(int[] arr) {
    int len = arr.length;

    buildHeap(arr, len);

    for (int i = len - 1; i > 0; i--) {
        swap(arr, 0, i);
        len--;
        heapify(arr, 0, len);
    }
    return arr;
}

private void buildHeap(int[] arr, int len) {
    for (int i = (len / 2 - 1); i >= 0; i--) {
        heapify(arr, i, len);
    }
}

private void heapify(int[] arr, int i, int len) {
    int left = 2 * i + 1;
    int right = 2 * i + 2;
    int largest = i;

    if (left < len && arr[left] > arr[largest]) {
        largest = left;
    }

    if (right < len && arr[right] > arr[largest]) {
        largest = right;
    }

    if (largest != i) {
        swap(arr, i, largest);
        heapify(arr, largest, len);
    }
}

private void swap(int[] arr, int i, int j) {
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}
