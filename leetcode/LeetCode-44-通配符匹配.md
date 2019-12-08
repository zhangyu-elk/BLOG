# LeetCode-44-通配符匹配

## 
```
给定一个字符串 (s) 和一个字符模式 (p) ，实现一个支持 '?' 和 '*' 的通配符匹配。

'?' 可以匹配任何单个字符。
'*' 可以匹配任意字符串（包括空字符串）。
两个字符串完全匹配才算匹配成功。

说明:

s 可能为空，且只包含从 a-z 的小写字母。
p 可能为空，且只包含从 a-z 的小写字母，以及字符 ? 和 *。
示例 1:

输入:
s = "aa"
p = "a"
输出: false
解释: "a" 无法匹配 "aa" 整个字符串。
示例 2:

输入:
s = "aa"
p = "*"
输出: true
解释: '*' 可以匹配任意字符串。
示例 3:

输入:
s = "cb"
p = "?a"
输出: false
解释: '?' 可以匹配 'c', 但第二个 'a' 无法匹配 'b'。
示例 4:

输入:
s = "adceb"
p = "*a*b"
输出: true
解释: 第一个 '*' 可以匹配空字符串, 第二个 '*' 可以匹配字符串 "dce".
示例 5:

输入:
s = "acdcb"
p = "a*c?b"
输入: false
```
来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/wildcard-matching
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解题思路

* 1 通配符匹配的问题最简单的就是用回溯算法, 但是这里使用动态规划来解决
* 2 动态转移规划方案:
	* 使用二维数组`f[m][n]`记录, `s`前`m`个字符和`p`前`n`个字符是否匹配
	* 状态转移方程: 
		* ① 如果`s[m]==p[n] || p[n]=='.'`, 那么`f[m][n]=f[m-1][n-1]`
		* ② 如果`p[n]=='*'`, 那么`f[m][n]=f[m][n-1] || f[m-1][n]`(匹配0个或者1个)
	* 初始状态: `f[i][0]=false`,`f[0][j]`这决定于`p`开头的`*`
	* 结果: `p[m][n]`
* 3 这其实也是在遍历所有的匹配情况, 从前缀子串所有匹配成功的情况下推断下一个(前面匹配失败后续比不可能成功)

## C++代码实现
```
class Solution {
public:
    bool isMatch(string s, string p) {
        int m = s.size();
        int n = p.size();

        if(m == 0)
        {
        	for(int i = 0; i < n; i++)
        	{
        		if(p[i] != '*')
        			return false;
        	}
        	return true;
        }
        if(n == 0)	return false;

        int m_1 = m + 1;
        int n_1 = n + 1;
        bool **f = new bool*[m_1];
        bool *buffer = new bool[m_1*n_1];
        for(int i = 0; i < m_1; i++)
        	f[i] = buffer + i*n_1;

        f[0][0] = true;
        for(int i = 1; i < m_1; i++)
        {
        	f[i][0] = false;
        }
        for(int j = 1; j < n_1; j++)
        {
			if(p[j-1] == '*') f[0][j] = true;
			else {
				for(; j < n_1; j++)
					f[0][j] = false;
				break;
			}
        }


        for(int i = 1; i < m_1; i++)
        {
        	for(int j = 1; j < n_1; j++)
        	{
    			if(p[j-1] == '*')
    			{
    				f[i][j] = f[i][j-1] || f[i-1][j];
    			}
    			else if(s[i-1]==p[j-1] || p[j-1] == '?')
    			{
    				f[i][j] = f[i-1][j-1];
    			}
    			else
    			{
    				f[i][j] = false;
    			}
        	}
        }
        return f[m][n];
    }
};
```

$\color{red}{注意: 我们需要考虑长度0而不是下标, 所以数组长度需要+1}$