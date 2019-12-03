# LeetCode-37-解数独

## 题目
编写一个程序，通过已填充的空格来解决数独问题。

一个数独的解法需遵循如下规则：

数字 1-9 在每一行只能出现一次。
数字 1-9 在每一列只能出现一次。
数字 1-9 在每一个以粗实线分隔的 3x3 宫内只能出现一次。
空白格用 '.' 表示。

![图片1](http://upload.wikimedia.org/wikipedia/commons/thumb/f/ff/Sudoku-by-L2G-20050714.svg/250px-Sudoku-by-L2G-20050714.svg.png)

一个数独。

![图片2](http://upload.wikimedia.org/wikipedia/commons/thumb/3/31/Sudoku-by-L2G-20050714_solution.svg/250px-Sudoku-by-L2G-20050714_solution.svg.png)

答案被标成红色。

Note:

给定的数独序列只包含数字 1-9 和字符 '.' 。
你可以假设给定的数独只有唯一解。
给定数独永远是 9x9 形式的。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/sudoku-solver
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解决思路
* 1 使用回溯算法, 但是单独的回溯算法可能无法通过时间空间栈限制;
* 2 添加限制条件, 当我们经过一个数后该方块和该横和该列都会有一个限制条件

### 限制条件
* 1 块的索引: (row/3)\*3 + (col/3)
* 2 使用数组提前保存记录数字的信息, 可以使用普通数组即可
* 3 注意由于使用引用进行传递, 如果递归计算失败需要把`borad`和限制条件回退


## C++算法实现
```
class Solution {
	bool core(vector<vector<char>>& board, int rows[9][10], int cols[9][10], int cells[9][10], int row, int col)
	{
		int m = (row/3*3) + col/3;
		int index = row * 9 + col + 1;
		int r = index / 9;
		int c = index % 9;

		if(index > 81)	
        {
            //res = board;
            return true;
        }

		if(board[row][col] != '.')
		{
			return core(board, rows, cols, cells, r, c);
		}
		else
		{
			for(int i = 1; i < 10; i++)
			{
				if(!rows[row][i] && !cols[col][i] && !cells[m][i])
				{
					rows[row][i] = cols[col][i] = cells[m][i] = 1;
					board[row][col] = i + '0';
					if(core(board, rows, cols, cells, row, col))
						return true;
					rows[row][i] = cols[col][i] = cells[m][i] = 0;
					board[row][col] = '.';
				}
			}
		}
        return false;
	}
public:
    void solveSudoku(vector<vector<char>>& board) {
       	int rows[9][10] = { 0 };
       	int cols[9][10] = { 0 };
       	int cells[9][10] = { 0 };

        for(int i = 0; i < board.size(); i++)
        {
        	for(int j = 0; j < board[i].size(); j++)
        	{
                if(board[i][j] == '.')  continue;
        		int n = board[i][j] - '0';
        		int m = (i/3*3) + j/3;
        		rows[i][n] = 1;
        		cols[j][n] = 1;
        		cells[m][n] = 1;
        	}
        }

        core(board, rows, cols, cells, 0, 0);
    }
};
```

## 总结

$\color{red}{回溯算法的基本使用很简单, 但是在LeetCode上的题目感觉如果暴力回溯的话很可能通不过的, 我们需要尽可能的通过一些限制条件进行优化.}$
