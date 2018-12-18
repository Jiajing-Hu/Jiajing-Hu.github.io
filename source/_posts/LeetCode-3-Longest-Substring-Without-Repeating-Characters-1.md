---
title: LeetCode 3. Longest Substring Without Repeating Characters
date: 2018-12-18 20:33:13
tags: leetcode
categories: leetcode
---



> Given a string, find the length of the longest substring without repeating characters.
>
> Example 1:
>
> Input: "abcabcbb"
> Output: 3 
> Explanation: The answer is "abc", with the length of 3. 
> Example 2:
>
> Input: "bbbbb"
> Output: 1
> Explanation: The answer is "b", with the length of 1.
> Example 3:
>
> Input: "pwwkew"
> Output: 3
> Explanation: The answer is "wke", with the length of 3. 
> ​             Note that the answer must be a substring, "pwke" is a subsequence and not a substring.


题目要求输出不包含重复字符的最长子串的长度

一般来说，设计到字符串的问题，都可以利用暴力破解来得出答案

就这题来说，如果直接判断，并且对每一个子串进行判断，并且比较长度。这样也可以得到问题的解，但是在面试中这显然不是一个好的解，因为它的时间复杂度达到了O(n * n)

就这题来讲，还有一种更好的解法，就是利用map,依次记录**上一个**重复字符出现的位置，在遍历字符串的过程中反复更新最长无重复子串的长度和上一次重复字符出现的位置，通过一次遍历既可得到结果。

> 例如：string str = "abcabcabc";res = 0;
> 刚开始我们可以设置start = -1,记为暂时没有重复的数字（-1是因为计算方便，毕竟大部分语言下标是从0开始的）
> 于是我们开始遍历字符串，并且维护一个map
> 如果第i个字符不在map中，那么我们将其插入map中，令map[str[i]] = i，如果在map中，则更新其value，并且更新start，标记最近一次重复的地点
> 并且在每一次遍历的过程中，都会计算一次当前长度len = (i - start); 由于start值为最近一次重复的地点，所以这里的len绝对不会包含重复字符
> 最后，比较res和len更新res既可

### C++代码
```c++
int lengthOfLongestSubstring(string s) {
    vector <int> dict (256,-1);
    int start = -1;
    int res = 0;
    for (int i = 0;i < s.length();++i)
    {
        if(dict[s[i]] > start) 
            start = dict[s[i]];
        dict[s[i]] = i;
        res = max(res,i - start);
    }
    return res;
}
```

### Java代码
```java
    public int lengthOfLongestSubstring(String s) {
        HashMap<Character, Integer> hash = new HashMap<Character, Integer>();
        int res = 0;
        int start = -1;
        for (int i = 0; i < s.length(); ++i) {
            if (hash.containsKey(s.charAt(i)))
                start = Math.max(start,hash.get(s.charAt(i)));
            hash.put(s.charAt(i), i);
            res = Math.max(res,i - start);
        }
        return res;
    }
```

### Golang代码
```go
func lengthOfLongestSubstring(s string) int {
	dict := make(map[uint8]int)
	res,start := 0,-1
	for i,_ := range s{
		j,ok := dict[s[i]]
		if ok {
			if j > start{
				start = j
			}
		}
		dict[s[i]] = i
		if i - start > res{
			res = i - start
		}
	}
	return res
}
```

### python代码
```python
class Solution:
    def lengthOfLongestSubstring(self, s):
        d = {}
        start = -1
        res = 0
        i = 0
        for ch in s:
            if ch in d:
                start = max(start, d[ch])
            d[ch] = i
            res = max(res, i - start)
            i += 1
        return res
```