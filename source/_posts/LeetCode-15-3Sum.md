---
title: LeetCode 15. 3Sum
date: 2018-11-29 13:41:57
tags: leetcode
catories: leetcode
---

> Given an array nums of n integers, are there elements a, b, c in nums such that a + b + c = 0? Find all unique triplets in the array which gives the sum of zero.
> 
> Note:
> 
> The solution set must not contain duplicate triplets.
> 
> Example:
> 
> Given array nums = [-1, 0, 1, 2, -1, -4],
> 
> A solution set is:
> [
>   [-1, 0, 1],
>   [-1, -1, 2]
> ]

题目要求,在一个数组内找到三个数使之的和为0,结果返回一个二维数组.并且要求不能存在重复解

不能使用重复解,在不动脑筋的情况下其实可以考虑使用set <vector <int>> 存储结果,并且便利每一种解法以此存入set,这种方式.最后返回的时候将其强制转换为vector <vector<int>>即可

这种方式当然能通过,但是并不是一个好的方法.第一是使用了强制转换.然后时间复杂度上,由于需要遍历每一种解法,此时的时间复杂度将会变为O(n^3)

如果在在线OJ系统中,这种基于暴力的方式在数据量不是很大的时候是能够通过的.但是如果这种题目出现在了面试中的话,这种解法肯定是拿不到满分的.

对于这个问题,我们可以先看另外一个问题---2Sum

要求在一个数组中,找到两个数字使之加起来和为某个定值target,并且要求时间复杂度低于O(nlgn)

暴力解法很简单

```c
vector <int> twoSum (vector <int> nums, int target)
{
    for (int i = 0;i < nums.size();++i)
        for (int j = i + 1;j < nums.size();++j)
            if(nums[i] == numss[j])
                return {nums[i],nums[j]};
}
```
但是这种解法不符合时间复杂度的要求.所以这里我们就需要使用另外一种解法-双指针法
首选,我们对数组进行排序,使之有序,同时利用两个指针分别指向数组的头尾进行搜索解,可以将时间复杂度降低到O(nlgn)
```c++
vector <int> twoSum (vector <int> nums, int target)
{
    sort(nums.begin(),nums.end());
    int left = 0, right = nums.size() - 1;
    while(left < right)
    {
        if (nums[left] + nums[right] == target)
            return {nums[left],nums[right]};
        esle if (nums[left] + nums[right] > target)
            --right;
        else if (nums[left] + nums[right] < target)
            ++left;
    }
}
```
利用这种思路,就可以延伸到3SUM这个问题上来.题目既然要求三个数之和等于0,那么就相当于要求两个数之和等于第三个数.根据这个思路.我们可以对数组遍历一次,对每个遍历的数字进行求2Sum操作即可.

### C++代码
```c++
vector<vector<int>> threeSum(vector<int>& nums) {
	sort (nums.begin(),nums.end());
	vector <vector<int>> res;
    if (nums.size() < 3)
        return res;
	for (int i = 0;i < nums.size() - 2;++i)
	{
		if (i && nums[i] == nums[i-1])
			continue;       //防止重复解
		int target = 0 - nums[i];
		int left = i + 1,right = nums.size() - 1;
		while(left < right)
		{
			if(nums[left] + nums[right] < target)
				++left;
			else if (nums[left] + nums[right] > target)
				--right;
			else 
			{
				res.push_back({nums[i],nums[left],nums[right]});
				++left,--right;
				while(left < nums.size() && nums[left] == nums[left - 1]) ++left;       //一个数可能有多组解,这里是算出其他的解
				while (right > i && nums[right] == nums[right +1 ]) --right;
			}
		}
	}
	return res;
}
```

### Java代码

```java
 public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> list = new ArrayList<>();
        if (nums.length < 3)
            return list;
        Arrays.sort(nums);
        for (int i = 0; i < nums.length - 2; ++i) {
            if (i > 0 && nums[i] == nums[i - 1])
                continue;
            int left = i + 1, right = nums.length - 1;
            while (left < right) {
                if (nums[left] + nums[right] < -nums[i])
                    ++left;
                else if (nums[left] + nums[right] > -nums[i])
                    --right;
                else {
                    List<Integer> l = new ArrayList<Integer>();
                    l.add(nums[i]);
                    l.add(nums[left++]);
                    l.add(nums[right--]);
                    list.add(l);
                    while (left < nums.length && nums[left] == nums[left - 1]) ++left;
                    while (right > i && nums[right] == nums[right + 1]) --right;
                }
            }
        }
        return list;
    }
```
