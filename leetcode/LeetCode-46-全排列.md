# LeetCode-46-全排列

## 题目
给定一个没有重复数字的序列，返回其所有可能的全排列。

示例:

输入: [1,2,3]
输出:
[
  [1,2,3],
  [1,3,2],
  [2,1,3],
  [2,3,1],
  [3,1,2],
  [3,2,1]
]

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/permutations
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解决思路

* 该题时非常经典的可以使用回溯算法解决的题目, 全排列
* 回溯算法的树图如下, 主要的注意点是在于对于错误情况的剪枝, 需要记录经过的路径.
![数](https://raw.githubusercontent.com/zhangyu012/picture_picgo/master/blogImg/20191219210221.png)

## C++实现
```
class Solution {
	void core(vector<int>& nums, vector<int> &track, vector<vector<int>> &res)
	{
		if(track.size() == nums.size())
		{
			res.push_back(track);
			return;
		}

		for(int i = 0; i < nums.size(); i++)
		{
			if(track.end() == find(track.begin(), track.end(), nums[i]))
			{
				track.push_back(nums[i]);
				core(nums, track, res);
				track.pop_back();
				//撤销
			}
		}
	}
public:
    vector<vector<int>> permute(vector<int>& nums) {
    	vector<vector<int>> res;
        vector<int> track;
        core(nums, track, res);
        return res;
    }
};
```