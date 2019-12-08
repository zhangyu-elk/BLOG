# LeetCode-32-最长有效括号

## 题目
给定一个只包含 '(' 和 ')' 的字符串，找出最长的包含有效括号的子串的长度。

示例 1:

输入: "(()"
输出: 2
解释: 最长有效括号子串为 "()"
示例 2:

输入: ")()())"
输出: 4
解释: 最长有效括号子串为 "()()"

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/longest-valid-parentheses
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解题思路

* 1 该题的标签为动态规划, 如果使用暴力计算的方式应当是也可以的.
* 2 状态转移:
	* 以一个长度为`n`的数组`D`来记录以该索引字符结尾的最长有效括号子串长度
	* 如果字符是`(`那么必然是0无需考虑, 只需考虑`)`的长度变化
	* 以代码的方式说明:
```
	if(data[i]==')' && i > 0)
	{
		if(data[i-1] == '(')
		{
			D[i] = 2+(i>1?D[i-2]:0);
		}
		else
		{
			if(data[i] == data[i-1-D[i-1]])
			{
				D[i] = 2 + D[i-1]];
			}
			else
			{
				D[i] = 0;
			}
		}
	}
	else
	{
		D[i]=0;
	}
```

## C++实现
```
class Solution {
public:
    int longestValidParentheses(string s) {
        int max = INT_MIN;
        int len = s.size();
        if(len <= 0)    return 0;

        int *D = new int[len];
        

        for(int i = 0; i < len; i++)
        {
            D[i] = 0;
        }

        for(int i = 0; i < len; i++)
        {
			if(s[i]==')' && i > 0)
			{
				if(s[i-1] == '(')
				{
					D[i] = 2+(i>1?D[i-2]:0);
				}
				else
				{
					if(i-1-D[i-1] >= 0 && s[i] != s[i-1-D[i-1]])
					{
						D[i] = 2 + D[i-1];
                        if(i-1-D[i-1] > 0)
                            D[i] += D[i-1-D[i-1]-1];
					}
					else
					{
						D[i] = 0;
					}
				}
			}
			else
			{
				D[i]=0;
			}        	
        }

        for(int i = 0; i < len; i++)
        	max = max < D[i] ? D[i] : max;

        delete [] D;
        return max;
    }
};
```
