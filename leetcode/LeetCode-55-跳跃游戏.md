# LeetCode-55-跳跃游戏

## 题目
给定一个非负整数数组，你最初位于数组的第一个位置。

数组中的每个元素代表你在该位置可以跳跃的最大长度。

判断你是否能够到达最后一个位置。

示例 1:

输入: [2,3,1,1,4]
输出: true
解释: 我们可以先跳 1 步，从位置 0 到达 位置 1, 然后再从位置 1 跳 3 步到达最后一个位置。
示例 2:

输入: [3,2,1,0,4]
输出: false
解释: 无论怎样，你总会到达索引为 3 的位置。但该位置的最大跳跃长度是 0 ， 所以你永远不可能到达最后一个位置。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/jump-game
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解题思路

* 如果每一个位置都是大于0的数, 那么必然可以达到数组最后一个位置
* 如果有位置为0, 那么则其前面的点能够跳过为0的位置: 
```
if d[i] = 0
	for(int j = 0; j < i; j++)
		if(d[j] > (i-j))
			break
```

## C++实现
```
class Solution {
public:
    bool canJump(vector<int>& nums) {
        if(nums.size() <= 1)    return true;
        for(int i = 0; i < nums.size()-1; i++)
        {
        	if(nums[i] == 0)
        	{
        		bool valid = false;
        		for(int j = 0; !valid && j < i; j++)
        		{
        			if(nums[j] > (i-j))
        			{
        				valid = true;
        			}
        		}
        		if(!valid)	return false;
        	}
        }
        return true;
    }
};
```