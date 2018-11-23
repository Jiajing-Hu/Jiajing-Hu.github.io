---
title: Leetcode 1. Two Sum
date: 2018-11-23 14:43:04
tags: leetcode
---

leetcode的第一题
也算是自己重新开始刷leetcode的一个开始吧。

先看题目：
> Given an array of integers, return indices of the two numbers such that they add up to a specific target.
>
>  You may assume that each input would have exactly one solution, and you may not use the same element twice.
>
>  Example:
>
>  Given nums = [2, 7, 11, 15], target = 9,
>
>  Because nums[0] + nums[1] = 2 + 7 = 9,
>  return [0, 1].

其实题目的要求也挺简单的，就是给一个数组和一个target，要求返回数组中两数和为target的两个值的下标，同时每个值只能使用一次，也就是说不能存在 nums[x] + nums[x] = target这种情况的出现。数据满足有且只有一个解；

首先看到这种题目，十分容易想到的一种方法就是暴力破解法，代码如下

```c++
class Solution {
public:
vector<int> twoSum(vector<int>& nums, int target) {
    vector <int> res;
    for (int i = 0; i < nums.size();++i)
        for (int j = i + 1;j < nums.size();++j)
            if(nums[i] + nums[j] == target)
            {
                res = {i,j};
                break;
            }
    return res;
}
};
```
这里使用了双重循环，判定每一种情况，观察是否能够满足条件。毫无疑问是能够得到结果的，但是并不是一个好的解，因为它的时间复杂度为O(n ^ 2)

一般来说，涉及到算法相关的事情，遇到这种时间复杂度的算法一般就要十分小心了，因为O(n ^ 2)时间复杂度是很容易导致TLE

实际上，这个解在leetcode上只能排名后10%。所以肯定是有更好的解的。

我们的目标是找到 target = nums[x] + numx[y]

那么nums[x]和nums[y]都是nums中的元素。那么每次我们判定一个值，如果满足输出条件，我们可以直接输出，如果不满足的话，可以在map中存储已经查询过的值，那么就会将时间复杂度优化到O(nlgn)。将会大大优化查询的时间复杂度，如果使用unordered_map的话，将会优化到O(n)时间复杂度

> map底层是红黑树，查询时间复杂度O(lgn)，unordered_map底层是哈希，查询时间复杂度为O(n)


代码如下
```c
vector<int> twoSum(vector<int>& nums, int target) {
    unordered_map <int,int> mp;         //使用哈希
    vector <int> res(2);
    mp[nums[0]] = 0;                    //因为题目规定一定有解，所以可以先导入第一个元素
    for (int i = 1;i < nums.size();++i)
    {
        int number = target - nums[i];      //目标数字
        if (mp.count(number))
        {
            res = {mp[number], i};          //如果存在，那么直接break
            break;
        }
        else 
            mp[nums[i]] = i;            //不存在的话，就将新的数据插入map中
    }
    return res;
}
```

这种方式牺牲了空间，但是时间复杂度大大减少了，时间从188ms减少到了4ms。从后10%到了前0.4%

> Runtime: 4 ms, faster than 99.96% of C++ online submissions for Two Sum.


另附Java解法：
```java
    public int[] twoSum(int[] nums, int target) {
        int[] res = new int[2];
        HashMap<Integer,Integer> map = new HashMap<Integer, Integer>();
        map.put(nums[0],0);
        for (int i = 1;i < nums.length;++i) {
           int number = target - nums[i];
           if(map.containsKey(number)){
               res[0] = map.get(number);
               res[1] = i;
               break;
           }
           else {
               map.put(nums[i],i);
           }
        }
        return res;
    }

```

这题算是一个简单的开始吧，哈希的简单应用。

希望自己以后也好好加油
