---
title: LeetCode 11.Container With Most Water
date: 2018-12-24 20:07:47
tags: leetcode
categories: leetcode
---

> Given n non-negative integers a1, a2, ..., an , where each represents a point at coordinate (i, ai). n vertical lines are drawn such that the two endpoints of line i is at (i, ai) and (i, 0). Find two lines, which together with x-axis forms a container, such that the container contains the most water.

> Note: You may not slant the container and n is at least 2.

经典的双指针，作为双指针的练习还是很不错的

### C++
```c++
class Solution {
public:
int maxArea(vector<int>& height) 
{
    int left = 0,right = height.size() - 1;
    int res = 0;
    while(left < right)
    {
        int min_height = min(height[left],height[right]);
        res = max(res,min_height * (right - left));
        while(height[left] <= min_height && left < right) ++left;
        while(height[right] <= min_height && left < right) --right;
    }
    return res;
}
};
```
### Java

```java
class Solution {
    public int maxArea(int[] height) {
        int left = 0, right = height.length - 1;
        int res = 0;
        while (left < right) {
            int min_height = height[left] < height[right] ? height[left] : height[right];
            res = res > min_height * (right - left) ? res : min_height * (right - left);
            while (height[left] <= min_height && left < right) ++left;
            while (height[right] <= min_height && left < right) --right;
        }
        return res;
    }
}
```
