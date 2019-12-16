# LeetCode-34-在排序数组中查找元素的第一个和最后一个位置

## 题目
给定一个按照升序排列的整数数组 nums，和一个目标值 target。找出给定目标值在数组中的开始位置和结束位置。

你的算法时间复杂度必须是 O(log n) 级别。

如果数组中不存在目标值，返回 [-1, -1]。

示例 1:

输入: nums = [5,7,7,8,8,10], target = 8
输出: [3,4]
示例 2:

输入: nums = [5,7,7,8,8,10], target = 6
输出: [-1,-1]

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解题思路

* 1 暴力遍历解法, 但是意义不大
* 2 采用二分法找到一个`target`, 然后分别往前往后分别获得`=target`的开头和结尾

* 3 以上二分法时间复杂的是`O(logn+X)`应当符合要求

## C++实现
```
class Solution {
	void core(vector<int>& nums, int l, int r, int target, vector<int>& res)
	{
		if(l > r)	return;

		int m = (l + r) / 2;
		if(nums[m] > target)
			core(nums, l, m - 1, target, res);
		else if(nums[m] < target)
			core(nums, m + 1, r, target, res);
		else
		{
			int t = m;
			while(t >= l && nums[t]==target)	t--;
			res[0] = (t+1);
			t = m;
			while(t <= r && nums[t]==target) t++;
			res[1] = (t-1);
		}
	}
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        vector<int> res(2, -1);
        core(nums, 0, nums.size()-1, target, res);
        return res;
    }
}; 
```