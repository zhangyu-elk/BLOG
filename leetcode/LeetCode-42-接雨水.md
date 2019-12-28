# LeetCode-42-接雨水

## 题目
给定 n 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

[图片](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/22/rainwatertrap.png)

上面是由数组 [0,1,0,2,1,0,1,3,2,1,2,1] 表示的高度图，在这种情况下，可以接 6 个单位的雨水（蓝色部分表示雨水）。 感谢 Marcos 贡献此图。

示例:

输入: [0,1,0,2,1,0,1,3,2,1,2,1]
输出: 6

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/trapping-rain-water
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解决思路

* 对于每一个点分别往左和右遍历, 找到左侧最大和右侧最大`maxL/maxR`, 那么该点能存储的水的数量为`max(maxL,marR)-D[i]`。 该种方式的时间复杂度为`O(N2)`, 如果做算法题的话通常采用暴力解法的意义不是特别大.

* 对于每一个点往前后遍历实际是存在重合的, 可以通过预先计算每个点的`maxL/maxR`数组; 最后可存储水的数量就是: `max(maxL[i],marR[i])-d[i]`的总和.

	* 求取`maxR[]`时可以从后往前遍历, 时间复杂度仅为O(n)

## C++实现
```
class Solution {
public:
    int trap(vector<int>& height) {
    	int len = height.size();
    	if(len <= 2)	return 0;
    	int *maxR = new int[len];
    	int *maxL = new int[len];

    	maxR[len-1] = height[len-1]; 
    	for(int i = len - 2; i >= 0; i--)
    	{
    		maxR[i] = max(height[i], maxR[i+1]);
    	}

    	maxL[0] = height[0];
    	for(int i = 1; i < len; i++)
    	{
    		maxL[i] = max(height[i], maxL[i-1]);
    	}

    	int res = 0;
    	for(int i = 1; i < len - 1; i++)
    	{
    		res += min(maxL[i],maxR[i]) - height[i];
    		//如果左侧或者右侧没有大于其的高度, maxL/maxR中存储的就是高度, 存储的水量就是0
    	}
        delete []maxR;
        delete []maxL;
    	return res;

    }
};
```