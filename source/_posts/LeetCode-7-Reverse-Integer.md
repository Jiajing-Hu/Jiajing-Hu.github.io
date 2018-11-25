---
title: LeetCode 7. Reverse Integer
date: 2018-11-24 20:41:23
tags: leetcode
categories: leetcode
---

> Given a 32-bit signed integer, reverse digits of an integer.
> 
> Example 1:
> 
> Input: 123
> Output: 321
> Example 2:
> 
> Input: -123
> Output: -321
> Example 3:
> 
> Input: 120
> Output: 21
> Note:
> Assume we are dealing with an environment which could only store integers within the 32-bit signed integer range: [−231,  231 − 1]. For the purpose of this problem, assume that your function returns 0 when the reversed integer overflows.

题目要求，将一个32位的整数进行翻转，并且返回的结果依然为一个32位整数。如果超过32位的范围，那么直接返回0

翻转整数本身十分简单，如下所示
```c
int Reverse (int x)
{
    int y = 0;
    while(x)
    {
        y = y * 10 + x % 10;
        x /= 10;
    }
    return y;
}
```
但是这个程序明显是有问题的，我们只要32位整数的范围是代表了除去符号位外一共有31位表示数字的值，所以它的范围便是[-2^31,2^31-1],我们以2^31-1为例，2^31 -1 = 2147483647,这里便是32位整数的上限。我们对其翻转得到的结果明显为7463847421，这里已经超过了32位整数的范围，所以将会导致结果溢出。

为了避免这个问题，我们可以使用如下程序

```c
    int reverse(int x) {
        long res = 0;
        while(x)
        {
            res = res * 10 + x % 10;
            x /= 10;
        }
        return (res > INT_MAX || res < INT_MIN) ? 0 : res;
    }
```
这个程序明显是对的，因为这里使用了long暂时存放了结果，并且在最后进行了一次结果的判断。

但是这里运用了类型的强制转换，有没有不使用强制转换的方法呢？答案是有的，我们自己在进行累加的时候，可以在循环中加入类型范围的判断就可以了。

32位整数的范围为-2147483648 - 2147483647(分别令其为INT_MIN,INT_MAX)

一旦res > INT_MAX / 10 或者 res < INT_MIN / 10的时候，如果继续对其进行res = res * 10操作，这里便会溢出，如果res == 恰好等于INT_MAX / 10,那么y值便不能大于7，res == INT_MIN / 10,res便不能小于 -8，

所以我们可以在每次累加的时候，对res的值进行一次判断，一旦发现溢出，立刻返回0

这种方式在累加的时候判断了数据的范围，可以在不使用强制转换的情况下防止溢出。同时，由于这里一旦发现溢出便不会继续往下而是直接return，程序的效率也会有所提升，从排名50%上升到了98%

代码如下

```c++
int reverse(int x) {
    int res = 0;
    while(x)
    {
        int y = x % 10;

        //这是一种明显溢出的情况
        //res = res * 10 + y;
        //res * 10部分就已经溢出了
        if (res > INT_MAX / 10 || res < INT_MIN / 10)
            return 0;
        // 当res * 10 恰好不溢出的时候，需要判断y
        // [- 2^31,2^31 -1]
        // 正负两侧的端点进行一次判断
        if ((res == INT_MAX / 10 && y > 7) || (res == INT_MIN / 10) && y < -8)
            return 0;
        res = res * 10 + y;
        x /= 10;
    }
    return res;
}

```

另附JAVA代码：

```java
    public int reverse(int x) {
        int res = 0;
        while (x != 0) {
            int y = x % 10;
            if (res > Integer.MAX_VALUE / 10 || res < Integer.MIN_VALUE / 10)
                return 0;
            else if((res == Integer.MAX_VALUE && y > 7) || (res == Integer.MIN_VALUE && y < -8))
                return 0;
            res = res * 10 + y;
            x /= 10;
        }
        return res;
    }
```
