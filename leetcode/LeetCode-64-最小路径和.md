# LeetCode-64-最小路径和

## 	题目
给定一个包含非负整数的 m x n 网格，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。

说明：每次只能向下或者向右移动一步。

示例:

输入:
[
  [1,3,1],
  [1,5,1],
  [4,2,1]
]
输出: 7
解释: 因为路径 1→3→1→1→1 的总和最小。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/minimum-path-sum
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解题思路

* 对于这种求路径的最简单的方式就是使用回溯算法, 遍历每一种情况

* 但是更高的方式是动态规划, 因为存在重复子问题: 比如对于`[1,3,5]/[1,1,5]`两种情况对于回溯算法都需要遍历下去, 实际上`[1,3,5]`这种情况必然大于`[1,1,5]`是不需要考虑下去的, 在这里可以进行一次合并.

### 动态规划思路

* 使用一个`m*n`的数组, 记录从左上角到该点的最小路径
* 遍历方式应当从左到右/从上到下的方式进行, 因为一个到一个点的路径长度决定于其上和左的点.

## C++实现

```
class Solution {
public:
    int minPathSum(vector<vector<int>>& grid) {
    	int m = grid.size();
    	if(m <= 0)	return 0;

    	int n = grid[0].size();
        int **f = new int*[m];
        int *buffer = new int[m*n];
        for(int i = 0; i < m; i++)
        {
        	f[i] = buffer + i*n;
        }
        f[0][0] = grid[0][0];
        for(int i = 1; i < m; i++)
        {
            f[i][0] = f[i-1][0] + grid[i][0];
        }
        for(int j = 1; j < n; j++)
        {
            f[0][j] = f[0][j-1] + grid[0][j];
        }
        for(int i = 1; i < m; i++)
        {
        	for(int j = 1; j < n; j++)
        	{
        		f[i][j] = grid[i][j] + min(f[i][j-1], f[i-1][j]);
        	}
        }
        int res = f[m-1][n-1];
        delete []f;
        delete []buffer;
        return res;
    }
};
```