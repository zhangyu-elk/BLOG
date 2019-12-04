# LeetCode-120-三角形的最小路径和

**[动态规划介绍]()**

## 题目
给定一个三角形，找出自顶向下的最小路径和。每一步只能移动到下一行中相邻的结点上。

例如，给定三角形：

[
     [2],
    [3,4],
   [6,5,7],
  [4,1,8,3]
]
自顶向下的最小路径和为 11（即，2 + 3 + 5 + 1 = 11）。

说明：

如果你可以只使用 O(n) 的额外空间（n 为三角形的总行数）来解决这个问题，那么你的算法会很加分。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/triangle
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解题思路

* 1 该题实际上是可以使用回溯算法解决的, 感觉就是涉及到多层遍历、树遍历的结构都可以使用回溯算法
* 2 本题 的标签是动态规划, 所以此次使用动态规划计算
* 3 学习过动态规划我们实际上在不涉及到额外的限制或者要求时, 我们实际上可以近使用`O(n)`的空间进行计算
```
N[1] = [2]
N[2] = [5, 6]
......
```
&emsp;&emsp;由于每一层的数据仅在其下一层计算时使用, 所以我们不需要保存每一层的数据. 到了`N[2](第二层)`那么第一层`N[1]`就可以舍弃了, 甚至我们直接在同一个数组上计算. $\color{red}{后续阶段有前阶段决定, 并且确定后就无需在关心之前的阶段}$

&emsp;&emsp;假设将数据保存在二维数组`D`中, 那么第N层第m个数: `D[N][m] = min(D[N-1][m-1], D[N-1][m])`; 第M个节点, 左上角为`m-1`, 右上角为`m+1`.$\color{red}{状态迁移方程}$


## C++实现
```
class Solution {
public:
    int minimumTotal(vector<vector<int>>& triangle) {
    	int n = triangle.size();
        int *data = new int[n];

        data[0] = triangle[0][0];
        for(int i = 1; i < n; i++)
        {
        	int l = triangle[i].size();
        	for(int j = l - 1; j >= 0; j--)
        	{
        		int left, right = INT_MAX;
        		int t = triangle[i][j];
        		if(j > 0)
        		{
        			left = data[j-1]
        		}
        		//左上角
        		if(j < triangle[i-1].size())
        		{
        			right = data[j];
        		}
        		//右上角
        		data[j] = (left < right) ? (left+t) : (right+t);
        	}
        }

        int min = INT_MAX;
        for(int i = 0; i < n; i++)
        {
        	if(min > data[i])	min = data[i];
        }
        delete data[];
    }
};
```
$\color{red}{上述判断大小写就是对重复子问题去重}$