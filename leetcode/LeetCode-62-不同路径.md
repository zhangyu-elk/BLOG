# LeetCode-62-不同路径

## 题目
一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为“Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为“Finish”）。

问总共有多少条不同的路径？

![图片](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/22/robot_maze.png)

例如，上图是一个7 x 3 的网格。有多少可能的路径？

说明：m 和 n 的值均不超过 100。

示例 1:

输入: m = 3, n = 2
输出: 3
解释:
从左上角开始，总共有 3 条路径可以到达右下角。
1. 向右 -> 向右 -> 向下
2. 向右 -> 向下 -> 向右
3. 向下 -> 向右 -> 向右
示例 2:

输入: m = 7, n = 3
输出: 28

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/unique-paths
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解题思路

* 该题需要遍历所有情况而不是求取是否能达到最终结果, 可以使用回溯算法


## C++实现
```
class Solution {
	void core(int x, int y, int m, int n, int &res)
	{
		if(x=m-1&&y=n-1) res++;

		if(x < m-1)
		{
			core(x+1, y, m, n, res);
		}
		if(y < n-1)
		{
			core(x,y+1,m,n,res);
		}
	}
public:
    int uniquePaths(int m, int n) {
        int res;
        core(0,0,m,n,res);
    }
};
```

$\color{res}{不过上述代码无法通过, 时间超时, 需要进行一定的优化}$

## C++优化实现
```
class Solution {
	int core(int x, int y, int m, int n, int **tmp)
	{
        if(x >= m || y >= n)  return 0;
		if(x==m-1&&y==n-1)	return 1;
		if(x <= m-1 && y <= n-1 && tmp[x][y] > 0)
			return tmp[x][y];

		int t1 = 0, t2 = 0;
		if(x < m-1)
		{
		    t1 = tmp[x+1][y] = core(x+1, y, m, n, tmp);
		}
		if(y < n-1)
		{
			t2 = tmp[x][y+1] = core(x,y+1,m,n,tmp);
		}
        return t1+t2;
	}
public:
    int uniquePaths(int m, int n) {
        if(m == 1 && n == 1)    return 1;
    	int **tmp = new int*[m];
    	int *buffer = new int[m*n];
    	for(int i = 0; i < m; i++)
    	{
    		tmp[i] = buffer + i*n;
            for(int j = 0; j < n; j++)
            {
                tmp[i][j] = -1;
            }
    	}
        //行是x,列是y
        int res = core(1,0,m,n, tmp) + core(0,1,m,n,tmp);
        delete []tmp;
        delete []buffer;
        return res;
    }
};
```

**优化后的算法能够通过!**