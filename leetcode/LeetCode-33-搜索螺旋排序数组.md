# LeetCode-33-搜索螺旋排序数组

## 题目
假设按照升序排序的数组在预先未知的某个点上进行了旋转。

( 例如，数组 [0,1,2,4,5,6,7] 可能变为 [4,5,6,7,0,1,2] )。

搜索一个给定的目标值，如果数组中存在这个目标值，则返回它的索引，否则返回 -1 。

你可以假设数组中不存在重复的元素。

你的算法时间复杂度必须是 O(log n) 级别。

示例 1:

输入: nums = [4,5,6,7,0,1,2], target = 0
输出: 4
示例 2:

输入: nums = [4,5,6,7,0,1,2], target = 3
输出: -1

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/search-in-rotated-sorted-array
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解题思路

* 对于普通的排序数组而言进行查找, 使用的往往就是二分法, 其效率也就是所需的`O(logn)`级别
* 对于螺旋排序数组而言, 如果使用暴力解法其时间复杂度就是`O(n)`, 我们依旧使用二分法:

### 螺旋数组的二分法

* 1 对于数组`[i,j]`而言, 可以分为两个递增的部分: `[i,k]/[k+1,j]`其中`d[k]>d[k+1]`, 对于示例中的数组就可以分为`[0,3]/[4,6]`. 注意: `d[i,k]>d[k+1,j]`。

* 2 取数组中点`m`, 有可能处于两部分的任意一部分; 如果`d[m]>d[i]`表示处于第一部分, 否则处于第二部分.

* 3 如果`d[m]>target`, 那么下一级需要遍历小于`d[m]`的集合:
	* 如果`d[m]`处于第一部分, 那么小于`d[m]`的集合分为两部分: `[i,m-1]`和`[k+1,j]`且`d[i,m-1]>d[k+1,j]`; 那么如果`d[i]>target`则考虑后一部分, 否则考虑前一部分.
	* 如果`d[m]`处于第二部分, 那么大于`d[m]`的部分仅为`[m+1,j]`了.

![图片1](https://raw.githubusercontent.com/zhangyu012/picture_picgo/master/blogImg/20191216203917.png)
```
∵ i=0,j=9
∴ m=4, d[m]=9

∵ d[i] < d[m]
∴ d[i,m-1] < d[m], 中点处于第一部分

∵ d[i] < target && d[k, j] < d[i]
∴ d[k, j] < target, 第二部分全部小于target忽略

∵ target < d[m]=9 && d[i,m-1] < d[m]
∴ next(i,m-1)

```
我们可以推导将范围缩减为`i,m-1`

* 4 如果`d[m]<target`, 和第三步进行同样的推导.

$\color{red}{这里的`[a,b]`指的是数据的集合.}$

### C++算法实现
```
class Solution {
	int core(vector<int> &nums, int l, int r, int target)
	{
		if(l == r)
			return nums[l] == target ? l : -1;
		if(l == r - 1)
		{
			if(nums[l] == target)	return l;
			if(nums[r] == target)	return r;
			return -1;
		}
        if(l > r)   return -1;


		int m = (r + l) / 2;
		if(nums[m]  > target)
		{
			if(nums[m] > nums[l] && nums[l] > target)
			{
				//第一部分
				return core(nums, m + 1, r, target);
			}
			else
			{
				//第二部分
				return core(nums, l, m - 1, target);
			}
		}
		else if(nums[m] < target)
		{
			if(nums[m] < nums[l] && nums[r] < target)
			{	
				return core(nums, l, m - 1, target);
			}
			else
			{
				return core(nums, m + 1, r, target);
			}
		}
		else
		{
			return m;
		}
	}
public:
    int search(vector<int>& nums, int target) {
        return core(nums, 0, nums.size()-1, target);
    }
};
```