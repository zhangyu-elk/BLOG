# LeetCode-31-下一个排列

## 题目
实现获取下一个排列的函数，算法需要将给定数字序列重新排列成字典序中下一个更大的排列。

如果不存在下一个更大的排列，则将数字重新排列成最小的排列（即升序排列）。

必须原地修改，只允许使用额外常数空间。

以下是一些例子，输入位于左侧列，其相应输出位于右侧列。
1,2,3 → 1,3,2
3,2,1 → 1,2,3
1,1,5 → 1,5,1

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/next-permutation
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

### 说明

**字典序**: 就是类似于英语字典的排序, 再排序当中高位大的优先; 那么该题其实就是计算给定数字序列组成的所有10进制数的大小排序, 找到指定的下一个.

## 解题思路

* 1 如果将传入的数组中的数字组成10进制数放入数组并排序, 我们要求解的就是当前顺序的下一个
```
[1,2,3]->[123,132,213,231,312,321]
```
如上, 如果传入的是`[2,3,1]`, 那么结果就是`[3,1,2]`

* 2 我们现在将传入的数组`[1,2,3]`视作数字`123`, 那么我们需要做的就是在这三位中交换某几位获得一个比`123`大, 但是比其他排序小的情况.

* 3 对于`[1,2,3]`, 可以交换`[1,3]/[2,3]/[1,2]`; 但是我们可以排除与高位`[1]`交换的情况, 因为如果低位就可以交换的话, 那么交换高位其获得的值肯定更大. `[2,3]/[1,2]`的交换肯定是前者更小, 因为后者改变的10进制的高位.

* 解决方案
	* ① 找到可交换的位置: 对于`[1,4,3,2]`, 我们的必然需要交换`[1]`, 因为在此之后是降序的, 进行交换必然小于当前数.
	* ② 对于以上`[1,4,3,2]`来说我们不需要考虑更高位的数字(其他数组可能更多), 首先`1`必须交换为后4位中最小的一个, 而其他数字必须以升序牌系列在其后.(大于`1432`的最小值, 即`2134`)。 处于优化考虑可以以快速排序来实现排序的操作.

##  C++实现
```
class Solution {
	//采用快速排序
	void sort(vector<int> &nums, int start, int end)
	{
		if(start >= end)	return;
		if(start + 1 == end)
		{
			if(nums[start] > nums[end])
				swap(nums[start],  nums[end]);
			return;
		}
		
		int h = end;
		int l = start + 1;
		while(l < h)
		{
			while(h > start && nums[h] >= nums[start])	h--;
			while(l <= end && nums[l] <= nums[start])  l++;
			if(l < h)
				swap(nums[h--], nums[l++]);
		}
		swap(nums[start], nums[h]);
		sort(nums, start, h-1);
		sort(nums, h+1, end);
	}
public:
    void nextPermutation(vector<int>& nums) {
    	int len = nums.size();
        if(len <= 1)    return ;

    	int l = -1;
        int min_index = 0;
        for(int i = len-1; i > 0; i--)
        {
        	if(nums[i] > nums[i-1])
        	{
        		l = i-1;
                for(int j = len - 1; j >= i; j--)
                {
                    if(nums[j] > nums[i-1])
                    {
                        swap(nums[j], nums[i-1]);
                        break;
                    }
                }      
        		break;
        	}
        }
        sort(nums, l+1, len-1);
        //后面的从小到大进行排序
    }
};
```



