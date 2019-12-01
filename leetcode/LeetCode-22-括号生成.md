# LeetCode-22-括号生成

## 题目
给出 n 代表生成括号的对数，请你写出一个函数，使其能够生成所有可能的并且有效的括号组合。

例如，给出 n = 3，生成结果为：

[
  "((()))",
  "(()())",
  "(())()",
  "()(())",
  "()()()"
]

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/generate-parentheses
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## C++实现
```
class Solution {
	/*
    *   t: 总队数
	*	l: 左括号个数
	*	r: 右括号个数
	*	str: 目前的字符串
	*	res: 保存结果的数组
	*/
	void core(int t, int l, int r, string str, vector<string> &res)
	{
        if(r > t || l > t)  return;

		if(r == l && t == l)
		{
			if(str.size())	res.push_back(str);	
		}

        if(l<=r)   
            core(t, l+1, r, str +"(", res);
        else
        {
            core(t, l+1, r, str + "(", res);	//左括号
			core(t, l, r+1, str + ")", res);	//右括号
        }
	}
public:
    vector<string> generateParenthesis(int n) {
    	string str;
    	vector<string> res;
        core(n, 0, 0, str, res);
        return res;
    }
};
```