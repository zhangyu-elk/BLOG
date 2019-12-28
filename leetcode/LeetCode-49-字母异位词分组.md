# LeetCode-49-字母异位词分组

## 题目
给定一个字符串数组，将字母异位词组合在一起。字母异位词指字母相同，但排列不同的字符串。

示例:

输入: ["eat", "tea", "tan", "ate", "nat", "bat"],
输出:
[
  ["ate","eat","tea"],
  ["nat","tan"],
  ["bat"]
]
说明：

所有输入均为小写字母。
不考虑答案输出的顺序。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/group-anagrams
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解决思路

* 暴力遍历解法, 两个字符串比较的时间复杂度是`O(2M)`; 将每个字符串和以排列的结果集进行比较, 对于第一个字符串来说结果集是空所以时间为`O(1)`, 对于最后一个字符串来说结果集最多可能有`N-1`个字符串数组于是时间复杂度达到了`O(2M*N)`; 此种解法的时间复杂度达到了`O(M*N2)`。

* `hash`的方式, 笔者最开始想到的是每一位对应的数字相加或者相乘, 但是似乎无法确保不重复; leetCode上的hash方式是采用质数相乘的方式进行, 因为质数仅能被自身整除所以这种方式是可行的.

* 比较简单的实现方式是先排序在使用`map`的方式.

## C++实现
```
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        map<string, vector<string>> map;
        vector<vector<string>> res;
        for(int i = 0; i < strs.size(); i++)
        {
        	string key = strs[i];
        	sort(key.begin(), key.end());
        	map[key].push_back(strs[i]);
        }

        for(auto i : map)
        {
        	res.push_back(i.second);
        }
        return res;
    }
};
```
