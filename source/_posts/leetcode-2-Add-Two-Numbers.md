---
title: leetcode 2. Add Two Numbers
date: 2018-11-24 02:56:03
tags: leetcode
categories: leetcode
---

废话少说，先看题目

> You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.
>
> You may assume the two numbers do not contain any leading zero, except the number 0 itself.
>
> Example:
>
> Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
> Output: 7 -> 0 -> 8
> Explanation: 342 + 465 = 807.
>

题目的要求很简单，让我们对两个数字进行相加。

但是这里有一点特殊的地方，那就是数字是倒序的方式存放在链表中，同时要求倒序输出我们得到的结果。

刚开始在这里倒序来倒序去的，一下子把自己绕进去了。其实根据负负得正的原理，居然要求我们相加后倒序输出，数字本身也是倒序存放的。那么我们直接对已经倒序存放的数字直接相加输出就可以了。

例如:
> 123 , 456
这两个数字分别代表了321和654。原数字相加的结果为975。那么我们需要输出为579。同时123 + 456 = 579

链表的相加其实就是类似于归并，跟归并排序的思想还是十分类似的。

我们以此对每一位进行想加和保存，同时需要注意进位，也就是要额外定义一个carry值来记录是否存在进位。

总体思路十分简单，下面是C++源码：

```c++
 public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode res = new ListNode(0);
        ListNode l = res;
        int carry = 0;
        while (l1 != null || l2 != null || carry != 0) {
            int cur = (l1 != null ? l1.val : 0) + (l2 != null ? l2.val : 0) + carry;
            carry = cur / 10;
            l.next = new ListNode(cur % 10);
            l = l.next;
            l1 = (l1 != null) ? l1.next : l1;
            l2 = (l2 != null) ? l2.next : l2;
        }
        return res.next;
    }
```


另附Java解法：
```java
 public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode res = new ListNode(0);
        ListNode l = res;
        int carry = 0;
        while (l1 != null || l2 != null || carry != 0) {
            int cur = (l1 != null ? l1.val : 0) + (l2 != null ? l2.val : 0) + carry;
            carry = cur / 10;
            l.next = new ListNode(cur % 10);
            l = l.next;
            l1 = (l1 != null) ? l1.next : l1;
            l2 = (l2 != null) ? l2.next : l2;
        }
        return res.next;
    }
```
