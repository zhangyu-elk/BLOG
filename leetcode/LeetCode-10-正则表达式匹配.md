# LeetCode-10-正则表达式匹配

## 题目说明
```
给你一个字符串 s 和一个字符规律 p，请你来实现一个支持 '.' 和 '*' 的正则表达式匹配。

'.' 匹配任意单个字符
'*' 匹配零个或多个前面的那一个元素
所谓匹配，是要涵盖 整个 字符串 s的，而不是部分字符串。

说明:

s 可能为空，且只包含从 a-z 的小写字母。
p 可能为空，且只包含从 a-z 的小写字母，以及字符 . 和 *。
示例 1:

输入:
s = "aa"
p = "a"
输出: false
解释: "a" 无法匹配 "aa" 整个字符串。
示例 2:

输入:
s = "aa"
p = "a*"
输出: true
解释: 因为 '*' 代表可以匹配零个或多个前面的那一个元素, 在这里前面的元素就是 'a'。因此，字符串 "aa" 可被视为 'a' 重复了一次。
示例 3:

输入:
s = "ab"
p = ".*"
输出: true
解释: ".*" 表示可匹配零个或多个（'*'）任意字符（'.'）。
示例 4:

输入:
s = "aab"
p = "c*a*b"
输出: true
解释: 因为 '*' 表示零个或多个，这里 'c' 为 0 个, 'a' 被重复一次。因此可以匹配字符串 "aab"。
示例 5:

输入:
s = "mississippi"
p = "mis*is*p*."
输出: false


来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/regular-expression-matching
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
```

## 解法思路

* 1 该题的标签为回溯算法，所以采用回溯算法来解决此题，[回溯算法介绍](https://www.jianshu.com/p/8dd10253714e)
* 2 我在回溯算法的文章中解决了一个类似的算法

## 注意
* 1 如果模式串长度为0的话, 那么匹配字符串也必须为空串
* 2 第一个字符为`*`的字符串应当属于错误的匹配串
* 3 如果连续两个`*`的话应当也属于错误的字符串
* 4 注意并不是`*`匹配零个或多个前面的那一个元素, 而是`A*`整体匹配零个或多个`A`.

## C++实现
```
class Solution {
private:
    bool isMatchCore(const char *s, const char *p)
    {
    	if (!*p)
        	return !*s;
        //如果模式串终结, 那么主串也必须结束; 反之不然

        bool fmatch = (*p == *s) || (*p == '.');

    	if(*(p+1)=='*')
    	{
            if(isMatchCore(s, p+2)) return true;
    		if(*s && fmatch) 
    		{
                return isMatchCore(s+1, p);
    		}
    	/* *号可以匹配多个, 但其实可以分解为这多种可能分别匹配0个字符的结果 */
    	}
    	else
    	{
    		if(!*s || !fmatch)	
    			return false;
    		else
    			return isMatchCore(s+1, p+1);
    	}

        return false;
    }
public:
    bool isMatch(string s, string p) {
        return isMatchCore(s.c_str(), p.c_str());
    }
};
```