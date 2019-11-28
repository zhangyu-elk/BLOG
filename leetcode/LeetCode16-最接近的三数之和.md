# LeetCode-16-最接近的三数之和

## 题目
给定一个包括 n 个整数的数组 nums 和 一个目标值 target。找出 nums 中的三个整数，使得它们的和与 target 最接近。返回这三个数的和。假定每组输入只存在唯一答案。

```
例如，给定数组 nums = [-1，2，1，-4], 和 target = 1.

与 target 最接近的三个数的和为 2. (-1 + 2 + 1 = 2).
```

## 解题思路

&emsp;&emsp;此题与`LeetCode-15-三数之和`的解法思路基本是一致的, 使用三个索引`i`/`min=i+1`/`max=len-1`; 如果总和大于`target`那么让`max--`(尽量靠拢`target`), 如果小于则`min++`, 如果等于直接返回`target`.

**注意**: 由于涉及到比较, 如果预先给`res`设置一个`INT_MAX`的话注意超出限制的情况.

## C++实现
```
class Solution {
public:
    int threeSumClosest(vector<int>& nums, int target) {
    	int res = 0;
    	bool flag = false;

        int len = nums.size();
        sort(nums.begin(), nums.end());
        for(int i = 0; i < len - 2; i++)
        {

        	int l = i + 1;
        	int r = len - 1;
        	while(l < r)
        	{
        		int total = nums[i] + nums[l] + nums[r];
        		int dis = total - target;
        		if(flag)
        		{
        			res = abs(dis) < abs(res - target) ? total : res;
        		}
        		else
        		{
        			flag = true
        			res = total;
        		}

        		if(dis > 0)
        		{
        			r--;		//距离大于0, 最大值往前移, 使得总和变小
        		}
        		else if(dis < 0)
        		{
        			l++;
        		}
        		else
        		{
        			return 0;
        		}
        	}
        }
        return res;
    }
};
```