# LeetCode-11-盛最多水的容器

## 题目
给定 n 个非负整数 a1，a2，...，an，每个数代表坐标中的一个点 (i, ai) 。在坐标内画 n 条垂直线，垂直线 i 的两个端点分别为 (i, ai) 和 (i, 0)。找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。

说明：你不能倾斜容器，且 n 的值至少为 2。

![图片](https://aliyun-lc-upload.oss-cn-hangzhou.aliyuncs.com/aliyun-lc-upload/uploads/2018/07/25/question_11.jpg)

图中垂直线代表输入数组 [1,8,6,2,5,4,8,3,7]。在此情况下，容器能够容纳水（表示为蓝色部分）的最大值为 49。


示例:

输入: [1,8,6,2,5,4,8,3,7]
输出: 49

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/container-with-most-water
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解题思路

* 1 该题其实就是求两条线之间的面积, 只不过长是两条线之间的距离, 高则是两条线中较短的那一条
* 2 解法1: 暴力遍历循环两遍获取最大值, 时间复杂度为`O(n)`

### 优化解法: 双指针法

&emsp;&emsp;在之前做过的一些题目中使用到了三指针法用于减少循环次数, 比较重要的一点是去掉不可能的情况, 能够在多种情况中选取最大或者较小的情况.

以上图为例, 对于数组`[0,8]`来说, 设`i=0;j=8`:
* 以暴力解法来说:
```
for i = 0, 7
	for j = i, 8
```

* 去掉不可能的情况, 减少遍历:

&emsp;&emsp;实际上对于`i=0;j=8`来说, 我们需要遍历这中的所有情况; 然而如果`N[j]<N[i]`, 那么此后所有以`j`为右边/`k>i`为左边的情况: 有`min(N[k],N[j])<=N[j]`和`j-k<j-i`, 其面积必然小于`{i,j}`的面积.

**总结**: 对于数组`[i,j]`, 如果`N[j]<N[i]`, 那么我们仅需考虑`{i,j}`和`[i,j-1]`; 反之亦然

ps: `[]`表示区间, `{}`表示左右下标对应的一种情况.

* 优化方案:

&emsp;&emsp;以双下标`i/j`分别指向前后, 遍历所有可能情况; 由于每次都是`i++/j--`两种情况, 其时间复杂度降低到了`O(N)`

### C++实现
```
class Solution {
public:
    int maxArea(vector<int>& height) {
        int l = 0;
        int r = height.size()-1;

        int maxArea = 0;
        while(l < r)
        {
        	int tmp = 0;
        	if(height[l]<height[r])
        	{
        		tmp = height[l++];
        	}
        	else
        	{
        		tmp = height[r--];
        	}
        	maxArea = maxArea > (tmp*(r-l+1)) ? maxArea : (tmp*(r-l+1));
        }
        return maxArea;
    }
};
```

