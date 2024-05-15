---
title: "Leetcode 工具类 构造输入"
date: 2024-05-12
tags: ["Java", "Algorithm"]
featuredImage: "https://s2.loli.net/2024/05/12/9jKdXJ8WaEq4it6.png"
featuredImagePreview: "https://s2.loli.net/2024/05/12/9jKdXJ8WaEq4it6.png"
draft: false
---

<!--more-->

### 代码

> 用于复制 Leetcode 的输入字符串，构造对应的数据结构

```java
package com.example.demo.leetcode;

import java.util.LinkedList;
import java.util.Queue;

public class LeetCodeUtil {

    private static final String NULL = "null";
    private static final String COMMA = ",";

    private LeetCodeUtil() {
    }


    static int[] newIntArray(String str) {
        str = format(str);
        if (str.isEmpty()) return new int[0];

        String[] parts = str.split(COMMA);
        int[] result = new int[parts.length];
        for (int i = 0; i < parts.length; i++) {
            result[i] = Integer.parseInt(parts[i].trim());
        }
        return result;
    }

    static int[][] new2DIntArray(String str) {
        str = format(str);
        if (str.isEmpty()) return new int[0][];
        String[] parts = str.replace("],", "]%%").split("%%");
        int[][] result = new int[parts.length][];
        for (int i = 0; i < parts.length; i++) {
            result[i] = newIntArray(parts[i]);
        }
        return result;
    }

    static String[] newStringArray(String str) {
        str = format(str);
        if (str.isEmpty()) return new String[0];

        String[] parts = str.split(COMMA);
        String[] result = new String[parts.length];
        for (int i = 0; i < parts.length; i++) {
            result[i] = parts[i].trim().replace("\"", "");
        }
        return result;
    }

    static ListNode newListNode(String str) {
        int[] input = newIntArray(str);
        if (input.length == 0) {
            return null;
        }
        ListNode root = new ListNode(input[0]);
        ListNode node = root;
        for (int i = 1; i < input.length; i++) {
            node.next = new ListNode(input[i]);
            node = node.next;
        }
        return root;
    }

    static TreeNode newTreeNode(String str) {
        String[] input = newStringArray(str);
        if (input.length == 0) {
            return null;
        }

        TreeNode root = new TreeNode(Integer.parseInt(input[0]));
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);

        for (int i = 1; i < input.length; i += 2) {
            TreeNode current = queue.poll();

            if (current != null) {
                if (!NULL.equals(input[i])) {
                    current.left = new TreeNode(Integer.parseInt(input[i]));
                    queue.offer(current.left);
                }
                if (i + 1 < input.length && !NULL.equals(input[i + 1])) {
                    current.right = new TreeNode(Integer.parseInt(input[i + 1]));
                    queue.offer(current.right);
                }
            }
        }
        return root;
    }

    private static String format(String str) {
        if (str != null) {
            str = str.replaceAll(" ", "");
            if (str.length() > 3) {
                return str.substring(1, str.length() - 1);
            }
        }
        return "";
    }
}

class ListNode {
    int val;
    ListNode next;

    ListNode() {
    }

    ListNode(int val) {
        this.val = val;
    }

    ListNode(int val, ListNode next) {
        this.val = val;
        this.next = next;
    }

    @Override
    public String toString() {
        return "{" +
                "val:" + val +
                ", next:" + next +
                '}';
    }
}

class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;

    TreeNode() {
    }

    TreeNode(int val) {
        this.val = val;
    }

    TreeNode(int val, TreeNode left, TreeNode right) {
        this.val = val;
        this.left = left;
        this.right = right;
    }

    @Override
    public String toString() {
        return "{" +
                "val:" + val +
                ", left:" + left +
                ", right:" + right +
                '}';
    }
}


```

