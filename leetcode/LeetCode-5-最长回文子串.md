# LeetCode-5-最长回文子串

## 题目
给定一个字符串 s，找到 s 中最长的回文子串。你可以假设 s 的最大长度为 1000。

示例 1：

输入: "babad"
输出: "bab"
注意: "aba" 也是一个有效答案。
示例 2：

输入: "cbbd"
输出: "bb"

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/longest-palindromic-substring
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解题思路
&emsp;&emsp;笔者不是按照顺序写下来的, 而是根据标签来的, 此题的标签在动态规划, 所以这里使用动态规划来解决. 最简单的方式就是逐个从中间开始往两侧推算以该点为中心的最长回文, 时间复杂度应当为`O(N2)`

&emsp;&emsp;动态规划重要的就是找到上一个阶段和下一个阶段之间的关系, 也就是状态转移方程. 回文字符串的关系就是, 假设`(i,j)`为回文字符串, 那么如果`D[i-1]=D[j+1]`则`(i-1,j+1)`也是回文字符串.

&emsp;&emsp;那么我们需要一个二位数组来记录数据, 空间复杂度应当为`O(N2)`, 时间复杂度也为`O(N2)`(填充该二维数组). 

&emsp;&emsp;由于我们是从短回文推长回文, 所有遍历方式应当是:

![这样的](https://i.loli.net/2019/12/05/r4Xw9UAzHYbFk7n.png)

**注意**: 由于`j>i`这个条件必须成立, 所以实际只需要一半的空间就可以存储所有的数据.

## C++解题思路
```
class Solution {
public:
    string longestPalindrome(string s) {
    	int n = s.size();
    	bool **data = new bool*[n];
    	bool *buffer = new bool[n*n];
    	for(int i = 0; i < n; i++)
    	{
    		data[i] = buffer + i*n;
    	}

    	int len = INT_MIN;
    	string res;
       

	    for(int k = 0; k < n; k++)
	    //我们应当从斜线计算, 计算长度为1、2、3...
	    {
	    	for(int i = 0; i < n; i++)
	    	{  
                if(i+k >= n)
	    		    break;

	    		if(k == 0)
	    		//长度为1, 必然是回文
	    		{
	    			data[i][i+k] = true;
	    		}
	    		else
	    		{
	    			if(s[i] == s[i+k])
	    			{
	    				//如果D[i]=D[j], 那么判断子串
	    				if(i+1<=i+k-1)
	    				//两个字符存在子串, 如果子串是回文则就是回文
	    				{
	    					data[i][i+k] = data[i+1][i+k-1];
	    				}
	    				else
	    				{
	    					//如果不存在子串, 则是2个字符的回文
	    					data[i][i+k] = true;
	    				}
	    			}
	    			else
	    			{
	    				data[i][i+k] = false;
	    			}		
	    		}
	    		if(data[i][i+k])
	    		{
	    			if(len < k)
	    			{
                        len = k;
	    				res = s.substr(i, k+1);
	    			}
	    		}
	    	}
	    }

        delete []buffer;
        delete []data;
        return res;
    }
};
```
