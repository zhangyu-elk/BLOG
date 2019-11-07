# 650. 只有两个键的键盘

## 题目
最初在一个记事本上只有一个字符 'A'。你每次可以对这个记事本进行两种操作：

Copy All (复制全部) : 你可以复制这个记事本中的所有字符(部分的复制是不允许的)。
Paste (粘贴) : 你可以粘贴你上一次复制的字符。
给定一个数字 n 。你需要使用最少的操作次数，在记事本中打印出恰好 n 个 'A'。输出能够打印出 n 个 'A' 的最少操作次数。

示例 1:

输入: 3
输出: 3
解释:
最初, 我们只有一个字符 'A'。
第 1 步, 我们使用 Copy All 操作。
第 2 步, 我们使用 Paste 操作来获得 'AA'。
第 3 步, 我们使用 Paste 操作来获得 'AAA'。
说明:

n 的取值范围是 [1, 1000] 。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/2-keys-keyboard
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解法思路

&emsp;&emsp;典型的动态规划问题, 分为两个动作: 1,复制;2,粘贴。 递归遍历这两种动作, 终止条件是 剩余个数小于等于0; 注意点是, 当前一次复制时后一次只能进行粘贴否则无意义, 当前无复制时只能进行复制否者无意义(避免死循环堆栈溢出).

## 代码
```

class Solution {
public:
	int min_step;
    int minSteps(int n) {
        min_step = INT_MAX;
        coreMinStep(1, 0, n - 1, 0, 0);
        return min_step;
    }
    //exist 已有  copy 复制的值  last 剩下的值 step 次数 action 动作
    void coreMinStep(int exist, int copy, int last, int step, int action)
    {
    	if(last < 0)
    	{
    		return ;
    	}
    	if(last == 0)
    	{
    		min_step = min(min_step, step);
    		return;
    	}
        // printf("%d %d %d %d %d\n", exist, copy, last, step, action);
    	//上一次进行了复制 
    	if(1 == action)
    	{
    		coreMinStep(exist + copy, copy, last - copy, step + 1, 0);
    	}
    	else if(0 == copy)
    	{
    		//之前没有进行复制
    		coreMinStep(exist, exist, last, step + 1, 1);
    	}
    	else
    	{
    		//复制  或者  粘贴
    		coreMinStep(exist, exist, last, step + 1, 1);
        	coreMinStep(exist + copy, copy, last - copy, step + 1, 0);
    	}
    }
};
``` 