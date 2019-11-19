# LeetCode-15-三数之和

## 题目
给定一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？找出所有满足条件且不重复的三元组。

注意：答案中不可以包含重复的三元组。

例如, 给定数组 nums = [-1, 0, 1, 2, -1, -4]，

满足要求的三元组集合为：
[
  [-1, 0, 1],
  [-1, -1, 2]
]

## 解题思路

* 1 数组中的数据可能重复, 但是要求的三元组不能重复; 所以首先进行排序
* 2 我们取三个索引, `i`用来遍历, `min=i+1`指向小值, `max=len-1`指向大值. 计算三个值的和, 如果大于0说明`max`太大了需要`max--`, 如果小于0说明小值太小了需要`min++`, 如果等于0说明正好合适. 
* 3 如果强行遍历, 其时间复杂度达到了`O(n3)`, 如上可以变为`O(n2)`



## 思路2的代码实现
```
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
     	vector<vector<int>> tmp;

     	int len = nums.size();
        sort(nums.begin(), nums.end());
     	for(int i = 0; i < len - 2; i++)
     	{
     		if(nums[i] > 0) break;		//后面的值更大
     		if(i > 0 && nums[i] == nums[i - 1]) continue;		//同样的值没必要重复计算

     		int l = i + 1;
     		int r = len - 1;
     		while(l < r)
     		{
     			int total = nums[i] + nums[l] + nums[r];
     			if(total < 0)
     			{
     				l++;
     			}
     			else if(total > 0)
     			{
     				r--;
     			}
     			else
     			{
     				tmp.push_back({nums[i], nums[r], nums[l]});
     				while(l < r && nums[l] == nums[++l]);
     				while(l < r && nums[r] == nums[--r]);
     			}
     		}
     	}
         return tmp;
    }
};

```
