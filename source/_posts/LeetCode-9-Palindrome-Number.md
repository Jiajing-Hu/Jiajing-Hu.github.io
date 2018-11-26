---
title: LeetCode 9. Palindrome Number
date: 2018-11-26 08:41:02
tags: leetcode
categories: leetcode
---

> Determine whether an integer is a palindrome. An integer is a palindrome when it reads the same backward as forward.
> 
> Example 1:
> 
> Input: 121
> Output: true
> Example 2:
> 
> Input: -121
> Output: false
> Explanation: From left to right, it reads -121. From right to left, it becomes 121-. Therefore it is not a palindrome.
> Example 3:
> 
> Input: 10
> Output: false
> Explanation: Reads 01 from right to left. Therefore it is not a palindrome.
> Follow up:
> 
> Coud you solve it without converting the integer to a string?
> 

题目要求判断是否回文，并且不能将数字转换为string

很简单直接贴代码

```c++

bool isPalindrome(int x)
{
    if (x < 0)
        return false;
    int palindrome = 0;
    int temp = x;
    while(temp)
    {
        palindrome = palindrome * 10 + temp % 10;
        temp /= 10;
    }
    return palindrome == x;
}
```

另外，其实转换为string的代码将会更加简洁（虽然题目不让使用）

```c++
bool isPalindrome(int x) {
    string str = to_string(x);
    return string(str.begin(),str.end()) == string(str.rbegin(),str.rend());
}
```

Java代码：

```java 
    public boolean isPalindrome(int x) {
        if(x < 0)
            return false;
        int palindrome = 0;
        int temp = x;
        while (temp != 0) {
            palindrome = palindrome * 10 + temp % 10;
            temp /= 10;
        }
        return palindrome == x;
    }

```

