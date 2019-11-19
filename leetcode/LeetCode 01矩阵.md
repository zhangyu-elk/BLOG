## LeetCode算法题

### 题目

给定一个由 0 和 1 组成的矩阵，找出每个元素到最近的 0 的距离。

两个相邻元素间的距离为 1 。

示例 1:
输入:

0 0 0
0 1 0
0 0 0
输出:

0 0 0
0 1 0
0 0 0
示例 2:
输入:

0 0 0
0 1 0
1 1 1
输出:

0 0 0
0 1 0
1 2 1
注意:

给定矩阵的元素个数不超过 10000。
给定矩阵中至少有一个元素是 0。
矩阵中的元素只在四个方向上相邻: 上、下、左、右

### 思路1
&emsp;&emsp;首先是中规中矩的思路, 对每一个`0`值做BFS广度优先遍历, 以下值实际代码
需要注意的是`matrix[x][y] <= matrix[index.first][index.second] + 1`这个判断条件, 假设原本某一个`0`BFS的层数小于或等于当前正在进行BFS所求的的值, 那么对于该点不再需要继续深入BFS了(因为该点的层数不变, 与之相关的点也不会改变)
```
class Solution {
public:
	vector<vector<int>> updateMatrix(vector<vector<int>>& matrix)
	{
		int mlen = matrix.size();
		int nlen = matrix[0].size();
		queue<pair<int, int>> queue;
		vector<vector<int>> dir = {{-1,0}, {0,-1}, {1,0},{0,1}};

		for(int i = 0; i < mlen; i++)
		{
			for(int j = 0; j < nlen; j++)
			{
				if(matrix[i][j] == 0)
				{
					queue.push(make_pair(i, j));
				}
				else
				{
					matrix[i][j] = 10001;
				}
			}
		}

		while(queue.size())
		{
			auto index = queue.front();
			queue.pop();
			for(auto c : dir)
			{
				int x = index.first + c[0];
				int y = index.second + c[1];
				if(x < 0 || y < 0 || x >= mlen || y >= nlen || matrix[x][y] <= matrix[index.first][index.second] + 1)
					continue;
				matrix[x][y] = matrix[index.first][index.second]; 
				queue.push(make_pair(x, y));
			}
		}

		return matrix;
	}
};
```

### 思路2(在leetcode上看到的)
&emsp;&emsp;对于一个点来说, 如果其值为0, 那么其层数为0; 如果为1, 其层数等于上下左右的最小值+1, 代码如下: 先对每个点的左上两个点进行比较, 再对右下两个点比较, 得出的就是最小的
```
class Solution_v2 {
public:
	vector<vector<int>> updateMatrix(vector<vector<int>>& matrix)
	{
		int mlen = matrix.size();
        int nlen = matrix[0].size();
		for(int i = 0; i < mlen; i++)
		{
			for(int j = 0; j < nlen; j++)
			{
				if(matrix[i][j])
				{
					int left = 10001, up = 100001;
					if(i > 0)
					{
						left = matrix[i-1][j] + 1;
					}
					if(j > 0)
					{
						up = matrix[i][j-1] + 1;
					}
					matrix[i][j] = min(left, up);
				}
				
			}
		}

		for(int i = mlen - 1; i >= 0; i--)
		{
			for(int j = nlen - 1; j >= 0; j--)
			{
				if(matrix[i][j])
				{
					int right = 100001, down = 100001;
					if(i < mlen - 1)
						right = matrix[i+1][j] + 1;
					if(j < nlen - 1)
						down = matrix[i][j+1] + 1;

					matrix[i][j] = min(matrix[i][j], min(right, down));
				}
			}
		}
		return matrix;
	}
};
```