# LeetCode-647-回文子串

## 题目
给定一个字符串，你的任务是计算这个字符串中有多少个回文子串。

具有不同开始位置或结束位置的子串，即使是由相同的字符组成，也会被计为是不同的子串。

示例 1:

输入: "abc"
输出: 3
解释: 三个回文子串: "a", "b", "c".
示例 2:

输入: "aaa"
输出: 6
说明: 6个回文子串: "a", "a", "a", "aa", "aa", "aaa".
注意:

输入的字符串长度不会超过1000。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/palindromic-substrings
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解题思路
&emsp;&emsp;使用动态规划求解[最长回文子串](https://www.jianshu.com/p/2ea10dd98ec5), 可以使用相同的规划思路遍历所有的情况.

&emsp;&emsp;使用一个二维数据记录数据, 坐标`[i,j]`分别指定回文子串的起始和结束位置; 如果`[i,j]`是回文子串, 那么当`d[i-1]=d[j+1]`可以推导`i-1,j+1`同样是回文子串


## C++实现
```
class Solution {
public:
    int countSubstrings(string s) {
    	int n = s.size();
    	bool **data = new bool*[n];
    	bool *buffer = new bool[n*n];
    	for(int i = 0; i < n; i++)
    	{
    		data[i] = buffer + i*n;
    	}

        int res = 0;       
	    for(int k = 0; k < n; k++)
	    //我们应当从斜线计算, 计算长度为1、2、3...
	    {
	    	for(int i = 0; i < n; i++)
	    	{  
                if(i+k >= n)
	    		    break;

	    		if(k == 0)
	    			data[i][i+k] = true;	//长度为1, 必然是回文
	    		else
	    		{
	    			if(s[i] == s[i+k])
	    			{
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
	    			res++;
	    		}
	    	}
	    }

        delete []buffer;
        delete []data;
        return res;
    }
};
```


