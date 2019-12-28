# LeetCode-70-爬楼梯

## 题目
假设你正在爬楼梯。需要 n 阶你才能到达楼顶。

每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？

注意：给定 n 是一个正整数。

示例 1：

输入： 2
输出： 2
解释： 有两种方法可以爬到楼顶。
1.  1 阶 + 1 阶
2.  2 阶
示例 2：

输入： 3
输出： 3
解释： 有三种方法可以爬到楼顶。
1.  1 阶 + 1 阶 + 1 阶
2.  1 阶 + 2 阶
3.  2 阶 + 1 阶

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/climbing-stairs
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解决思路

* 遍历所有情况使用回溯算法

## C++实现
```
class Solution {
	void core(int n, int &res)
	{
		if(n <= 1)	res++;
		if(n > 1)
		{
			core(n-1, res);
			core(n-2, res);
		}
	}
public:
    int climbStairs(int n) {
    	if(n <= 0)	return 0;
    	int res = 0;
        core(n, res);
    }
};
```

$\color{red}{以上这种解法,在44时超出了时间, 所以需要优化一下; 回溯算法很多时候可以使用数据存储结果以减少遍历, 也可以使用动态规划}$

## 动态规划思路

* `f[n] = f[n-1] + f[n-2]`

### C++实现
```
class Solution {
public:
    int climbStairs(int n) {
        if(n <= 1)  return 1;
        
        int *tmp = new int[n+1];
        tmp[1] = 1;
        tmp[2] = 2;
    	for(int i = 3; i <= n; i++)
    	{
    		if(i > 0)
    			tmp[i] = tmp[i-1];
    		if(i > 1)
    			tmp[i] += tmp[i-2];
    	}
        int res = tmp[n];
        delete [] tmp;
        return res;
    }
};
```