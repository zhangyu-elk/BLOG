# LeetCode-48-旋转图像

## 题目
给定一个 n × n 的二维矩阵表示一个图像。

将图像顺时针旋转 90 度。

说明：

你必须在原地旋转图像，这意味着你需要直接修改输入的二维矩阵。请不要使用另一个矩阵来旋转图像。

示例 1:

给定 matrix = 
[
  [1,2,3],
  [4,5,6],
  [7,8,9]
],

原地旋转输入矩阵，使其变为:
[
  [7,4,1],
  [8,5,2],
  [9,6,3]
]
示例 2:

给定 matrix =
[
  [ 5, 1, 9,11],
  [ 2, 4, 8,10],
  [13, 3, 6, 7],
  [15,14,12,16]
], 

原地旋转输入矩阵，使其变为:
[
  [15,13, 2, 5],
  [14, 3, 4, 1],
  [12, 6, 8, 9],
  [16, 7,10,11]
]

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/rotate-image
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解题思路

&emsp;&emsp;题目要求必须在原地进行旋转矩阵, 所以我们需要一步一步的进行旋转
![示例](https://raw.githubusercontent.com/zhangyu012/picture_picgo/master/blogImg/20191219214252.png)

* 一个长高为`n`的矩阵, 一共可以分为`n/2`圈, 矩阵旋转时每个圈各自分别进行旋转。
* 如图, 一个圈可以分为上下左右四部分, 左部分仅需往上移动一格, 其他三部分也仅需要往某一个方位移动一格; 往外扩展的部分也是相同:
![扩展](https://raw.githubusercontent.com/zhangyu012/picture_picgo/master/blogImg/20191219215356.png)

**使用伪代码表示即:**
```
if x <= y && n-1-y > x
	up
if x < y && n-1-y <= x
	right
if x > y && n-1-x >=y 
	left
if x >= y && n-1-x < y
	down
``` 

* 最后一共需要移动n次, 不过这样的时间复杂度达到了`O(N3)`的级别.

## C++实现
```
class Solution {
	void rswap(vector<vector<int>>& matrix,int x, int y, int last, int n)
	{
		int tmp = matrix[x][y];
		matrix[x][y] = last;
        cout<<x<<" "<<y<<endl;
		if(x == y && x < n/2)  
            return;

		if(x <= y && n-1-y > x)
		{
			rswap(matrix, x, y+1, tmp, n);
		}
		else if(x < y && n-1-y <= x)
		{
			rswap(matrix, x+1, y, tmp, n);
		}
		else if(x > y && n-1-x >= y)
		{
			rswap(matrix, x-1, y, tmp, n);
		}
		else if(x >= y && n-1-x < y)
		{
			rswap(matrix, x, y-1, tmp, n);
		}
	}
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size(); 
       	if(n <= 1)	return;
        for(int i = 0; i < n / 2; i++)
            for(int k = 0; k < n-2*i-1; k++)
                rswap(matrix, i, i+1, matrix[i][i], n);
    }
};
```


## 优化思路

以上的解法, 每一个点都需要移动`1~n`次, 我们实际可以直接算出来其应当转移的位置:
![图片](https://raw.githubusercontent.com/zhangyu012/picture_picgo/master/blogImg/20191219225724.png)
```
class Solution {

	void rswap(vector<vector<int>>& matrix, int x)
	{
		int n = matrix.size();
		//四条边
		for(int y = x; y < n-1-x; y++)
		{
			int t1 = matrix[x][y];
			int nx = y;
			int ny = n-1-x;
			int t2 = matrix[nx][ny];
			matrix[nx][ny] = t1;
			//右侧边的y是固定的为 n-1-x
			t1 = t2;

			nx = n-1-x;
			ny = n-1-y;
			t2 = matrix[nx][ny];
			matrix[nx][ny] = t1;
			//底边的x是固定的为 n-1-x
			t1 = t2;

			nx = n-1-y;
			ny = x;
			t2= matrix[nx][ny];
			matrix[nx][ny] = t1;
			//左侧边的y是固定的为 x

			matrix[x][y] = t2;
		}

	}
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size(); 
       	if(n <= 1)	return;
        for(int i = 0; i < n / 2; i++)
                rswap(matrix, i);
    }
};
```

ps: 从leetcode的测试结果来看, 优化很明显